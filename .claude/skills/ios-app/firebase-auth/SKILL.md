---
description: Load when implementing Firebase Auth with Google, Apple, or email sign-in in a SwiftUI iOS app.
---

# Firebase Auth Sub-Skill

> Load this skill when implementing authentication, user sign-in flows, OAuth providers, or managing user sessions in iOS apps.

## Core Principle

**NEVER build custom authentication UI or logic.** Use FirebaseUI for all sign-in flows. It handles 99% of edge cases you haven't thought of.

## Setup Checklist

### 1. Firebase Console Setup

1. Create Firebase project at console.firebase.google.com
2. Add iOS app (use your bundle identifier)
3. Download `GoogleService-Info.plist`
4. Enable authentication providers in Firebase Console:
   - Authentication → Sign-in method
   - Enable Google, Apple, Email/Password

### 2. Xcode Project Setup

Add to your `Info.plist`:

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <!-- Get this from GoogleService-Info.plist: REVERSED_CLIENT_ID -->
            <string>com.googleusercontent.apps.123456789</string>
        </array>
    </dict>
</array>

<key>GIDClientID</key>
<string>YOUR_CLIENT_ID.apps.googleusercontent.com</string>
```

### 3. Swift Package Dependencies

```swift
dependencies: [
    .package(url: "https://github.com/firebase/firebase-ios-sdk", from: "10.0.0")
]

// In target dependencies:
.product(name: "FirebaseAuth", package: "firebase-ios-sdk"),
.product(name: "FirebaseAuthUI", package: "firebase-ios-sdk"),
.product(name: "GoogleSignIn", package: "GoogleSignIn-iOS")
```

## FirebaseUI Integration

### Basic Setup

```swift
import SwiftUI
import FirebaseCore
import FirebaseAuth
import GoogleSignIn

@main
struct YourApp: App {
    init() {
        FirebaseApp.configure()
    }

    var body: some Scene {
        WindowGroup {
            RootView()
        }
    }
}
```

### Auth State Manager

```swift
import FirebaseAuth
import Combine

@Observable
class AuthenticationManager {
    var user: User?
    var isAuthenticated: Bool {
        user != nil
    }

    private var authStateHandler: AuthStateDidChangeListenerHandle?

    init() {
        configureAuthStateListener()
    }

    deinit {
        if let handle = authStateHandler {
            Auth.auth().removeStateDidChangeListener(handle)
        }
    }

    private func configureAuthStateListener() {
        authStateHandler = Auth.auth().addStateDidChangeListener { [weak self] _, user in
            self?.user = user
        }
    }

    func signOut() {
        do {
            try Auth.auth().signOut()
        } catch {
            print("Error signing out: \(error.localizedDescription)")
        }
    }
}
```

### Google Sign-In Implementation

```swift
import SwiftUI
import GoogleSignIn
import FirebaseAuth

struct GoogleSignInButton: View {
    @Environment(AuthenticationManager.self) private var authManager
    @State private var isLoading = false
    @State private var errorMessage: String?

    var body: some View {
        VStack {
            Button {
                signInWithGoogle()
            } label: {
                HStack {
                    Image(systemName: "globe")
                    Text("Sign in with Google")
                }
                .frame(maxWidth: .infinity)
                .padding()
                .background(Color.blue)
                .foregroundColor(.white)
                .cornerRadius(10)
            }
            .disabled(isLoading)

            if let error = errorMessage {
                Text(error)
                    .foregroundColor(.red)
                    .font(.caption)
            }
        }
    }

    private func signInWithGoogle() {
        guard let clientID = FirebaseApp.app()?.options.clientID else {
            errorMessage = "Missing client ID"
            return
        }

        let config = GIDConfiguration(clientID: clientID)
        GIDSignIn.sharedInstance.configuration = config

        guard let windowScene = UIApplication.shared.connectedScenes.first as? UIWindowScene,
              let rootViewController = windowScene.windows.first?.rootViewController else {
            errorMessage = "Cannot find root view controller"
            return
        }

        isLoading = true
        errorMessage = nil

        GIDSignIn.sharedInstance.signIn(withPresenting: rootViewController) { result, error in
            isLoading = false

            if let error = error {
                errorMessage = error.localizedDescription
                return
            }

            guard let user = result?.user,
                  let idToken = user.idToken?.tokenString else {
                errorMessage = "Failed to get user token"
                return
            }

            let credential = GoogleAuthProvider.credential(
                withIDToken: idToken,
                accessToken: user.accessToken.tokenString
            )

            Auth.auth().signIn(with: credential) { authResult, error in
                if let error = error {
                    errorMessage = error.localizedDescription
                }
            }
        }
    }
}
```

### Apple Sign-In Implementation

```swift
import SwiftUI
import AuthenticationServices
import FirebaseAuth
import CryptoKit

struct AppleSignInButton: View {
    @State private var currentNonce: String?
    @State private var errorMessage: String?

    var body: some View {
        VStack {
            SignInWithAppleButton { request in
                let nonce = randomNonceString()
                currentNonce = nonce
                request.requestedScopes = [.fullName, .email]
                request.nonce = sha256(nonce)
            } onCompletion: { result in
                switch result {
                case .success(let authorization):
                    handleAppleSignIn(authorization)
                case .failure(let error):
                    errorMessage = error.localizedDescription
                }
            }
            .signInWithAppleButtonStyle(.black)
            .frame(height: 50)

            if let error = errorMessage {
                Text(error)
                    .foregroundColor(.red)
                    .font(.caption)
            }
        }
    }

    private func handleAppleSignIn(_ authorization: ASAuthorization) {
        guard let appleIDCredential = authorization.credential as? ASAuthorizationAppleIDCredential,
              let nonce = currentNonce,
              let appleIDToken = appleIDCredential.identityToken,
              let idTokenString = String(data: appleIDToken, encoding: .utf8) else {
            errorMessage = "Failed to get Apple ID token"
            return
        }

        let credential = OAuthProvider.credential(
            withProviderID: "apple.com",
            idToken: idTokenString,
            rawNonce: nonce
        )

        Auth.auth().signIn(with: credential) { authResult, error in
            if let error = error {
                errorMessage = error.localizedDescription
            }
        }
    }

    // MARK: - Helpers

    private func randomNonceString(length: Int = 32) -> String {
        precondition(length > 0)
        var randomBytes = [UInt8](repeating: 0, count: length)
        let errorCode = SecRandomCopyBytes(kSecRandomDefault, randomBytes.count, &randomBytes)
        if errorCode != errSecSuccess {
            fatalError("Unable to generate nonce. SecRandomCopyBytes failed with OSStatus \(errorCode)")
        }

        let charset: [Character] = Array("0123456789ABCDEFGHIJKLMNOPQRSTUVXYZabcdefghijklmnopqrstuvwxyz-._")
        let nonce = randomBytes.map { byte in
            charset[Int(byte) % charset.count]
        }
        return String(nonce)
    }

    private func sha256(_ input: String) -> String {
        let inputData = Data(input.utf8)
        let hashedData = SHA256.hash(data: inputData)
        return hashedData.compactMap { String(format: "%02x", $0) }.joined()
    }
}
```

### Email/Password Sign-In

```swift
import SwiftUI
import FirebaseAuth

struct EmailSignInView: View {
    @State private var email = ""
    @State private var password = ""
    @State private var isLoading = false
    @State private var errorMessage: String?

    var body: some View {
        VStack(spacing: 16) {
            TextField("Email", text: $email)
                .textContentType(.emailAddress)
                .autocapitalization(.none)
                .textFieldStyle(.roundedBorder)

            SecureField("Password", text: $password)
                .textContentType(.password)
                .textFieldStyle(.roundedBorder)

            Button("Sign In") {
                signIn()
            }
            .disabled(isLoading || email.isEmpty || password.isEmpty)

            Button("Create Account") {
                createAccount()
            }
            .disabled(isLoading || email.isEmpty || password.isEmpty)

            if let error = errorMessage {
                Text(error)
                    .foregroundColor(.red)
                    .font(.caption)
            }
        }
        .padding()
    }

    private func signIn() {
        isLoading = true
        errorMessage = nil

        Auth.auth().signIn(withEmail: email, password: password) { result, error in
            isLoading = false
            if let error = error {
                errorMessage = error.localizedDescription
            }
        }
    }

    private func createAccount() {
        isLoading = true
        errorMessage = nil

        Auth.auth().createUser(withEmail: email, password: password) { result, error in
            isLoading = false
            if let error = error {
                errorMessage = error.localizedDescription
            }
        }
    }
}
```

## Account Linking

Handle the case where a user signs in with different providers for the same email:

```swift
func linkAccount(with credential: AuthCredential) {
    guard let currentUser = Auth.auth().currentUser else { return }

    currentUser.link(with: credential) { authResult, error in
        if let error = error as NSError? {
            if error.code == AuthErrorCode.credentialAlreadyInUse.rawValue {
                // Account already exists, merge data if needed
                print("Account already linked")
            } else {
                print("Error linking account: \(error.localizedDescription)")
            }
        }
    }
}
```

## Token Management

### Get Current User Token

```swift
func getCurrentUserToken() async throws -> String {
    guard let user = Auth.auth().currentUser else {
        throw NSError(domain: "Auth", code: -1, userInfo: [NSLocalizedDescriptionKey: "No user signed in"])
    }

    return try await user.getIDToken()
}
```

### Refresh Token

```swift
func refreshToken() async throws -> String {
    guard let user = Auth.auth().currentUser else {
        throw NSError(domain: "Auth", code: -1, userInfo: [NSLocalizedDescriptionKey: "No user signed in"])
    }

    return try await user.getIDToken(forcingRefresh: true)
}
```

## Common Gotchas

### 1. Keychain Access

Add keychain sharing capability if you need to share auth state across apps:

```
Capabilities → Keychain Sharing → + keychain group
```

### 2. App Delegate Setup

If using UIKit or AppDelegate:

```swift
import UIKit
import FirebaseCore
import GoogleSignIn

class AppDelegate: UIResponder, UIApplicationDelegate {
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        FirebaseApp.configure()
        return true
    }

    func application(_ app: UIApplication,
                     open url: URL,
                     options: [UIApplication.OpenURLOptionsKey: Any] = [:]) -> Bool {
        return GIDSignIn.sharedInstance.handle(url)
    }
}
```

### 3. Info.plist Scheme

Always verify `REVERSED_CLIENT_ID` matches your GoogleService-Info.plist:

```bash
# Get the value from GoogleService-Info.plist
grep REVERSED_CLIENT_ID GoogleService-Info.plist
```

### 4. Capability Setup for Apple Sign-In

Enable "Sign in with Apple" capability in Xcode:

```
Signing & Capabilities → + Capability → Sign in with Apple
```

### 5. Anonymous Auth Upgrade

Convert anonymous users to permanent accounts:

```swift
func upgradeAnonymousUser(with credential: AuthCredential) {
    guard let currentUser = Auth.auth().currentUser,
          currentUser.isAnonymous else { return }

    currentUser.link(with: credential) { authResult, error in
        if let error = error {
            print("Error upgrading anonymous user: \(error.localizedDescription)")
        } else {
            print("Anonymous user upgraded successfully")
        }
    }
}
```

## Security Best Practices

1. **Never store passwords** - Let Firebase handle it
2. **Use biometric auth** - Add Face ID/Touch ID on top of Firebase Auth
3. **Validate tokens server-side** - Always verify tokens in your backend
4. **Handle token expiration** - Refresh tokens proactively
5. **Implement proper logout** - Clear all local state on sign out

## Testing

### Simulator Testing

- Email/Password: Works in simulator
- Google Sign-In: Works in simulator
- Apple Sign-In: Requires real device for full testing

### Sandbox vs Production

Firebase Auth doesn't have a sandbox mode. Use separate Firebase projects for dev/staging/prod.

## Complete Auth Flow Example

```swift
import SwiftUI
import FirebaseAuth

@main
struct MyApp: App {
    @State private var authManager = AuthenticationManager()

    var body: some Scene {
        WindowGroup {
            Group {
                if authManager.isAuthenticated {
                    MainTabView()
                } else {
                    SignInView()
                }
            }
            .environment(authManager)
        }
    }
}

struct SignInView: View {
    var body: some View {
        VStack(spacing: 20) {
            Text("Welcome")
                .font(.largeTitle)
                .bold()

            GoogleSignInButton()
            AppleSignInButton()

            NavigationLink("Or sign in with email") {
                EmailSignInView()
            }
        }
        .padding()
    }
}
```

## When to Use What

- **Google Sign-In**: Most apps (widely used, easy)
- **Apple Sign-In**: Required if you offer any other social sign-in
- **Email/Password**: When users prefer email, or as fallback
- **Anonymous Auth**: For apps that need to work without sign-up
- **Phone Auth**: For phone-number-based apps (SMS verification)

## Bottom Line

Use FirebaseUI. Don't build custom auth UI. Don't roll your own token management. Focus on your app's unique features.
