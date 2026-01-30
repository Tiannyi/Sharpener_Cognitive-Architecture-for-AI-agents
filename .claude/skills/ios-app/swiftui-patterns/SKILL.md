# SwiftUI Patterns Sub-Skill

> Load this skill when architecting SwiftUI views, managing state, implementing navigation, or organizing SwiftUI code.

## Core Principle

**Use SwiftUI's native patterns.** Don't fight the framework. @Observable for models, NavigationStack for navigation, environment for dependency injection.

## MVVM with @Observable

### The Pattern

SwiftUI + @Observable (iOS 17+) = Modern MVVM without boilerplate.

```swift
import SwiftUI
import Observation

// Model
struct User: Identifiable, Codable {
    let id: String
    var name: String
    var email: String
}

// ViewModel
@Observable
class UserProfileViewModel {
    var user: User?
    var isLoading = false
    var errorMessage: String?

    private let repository: UserRepository

    init(repository: UserRepository) {
        self.repository = repository
    }

    func loadUser(id: String) {
        isLoading = true
        errorMessage = nil

        Task {
            do {
                user = try await repository.fetchUser(id: id)
            } catch {
                errorMessage = error.localizedDescription
            }
            isLoading = false
        }
    }

    func updateName(_ name: String) {
        user?.name = name
    }

    func saveUser() async {
        guard let user = user else { return }

        isLoading = true
        errorMessage = nil

        do {
            try await repository.updateUser(user)
        } catch {
            errorMessage = error.localizedDescription
        }
        isLoading = false
    }
}

// View
struct UserProfileView: View {
    @State private var viewModel: UserProfileViewModel
    let userId: String

    init(userId: String, repository: UserRepository) {
        self.userId = userId
        self._viewModel = State(wrappedValue: UserProfileViewModel(repository: repository))
    }

    var body: some View {
        VStack {
            if viewModel.isLoading {
                ProgressView()
            } else if let user = viewModel.user {
                Form {
                    TextField("Name", text: Binding(
                        get: { user.name },
                        set: { viewModel.updateName($0) }
                    ))

                    Text(user.email)
                        .foregroundColor(.secondary)

                    Button("Save") {
                        Task {
                            await viewModel.saveUser()
                        }
                    }
                }
            } else if let error = viewModel.errorMessage {
                ErrorView(message: error) {
                    viewModel.loadUser(id: userId)
                }
            }
        }
        .task {
            viewModel.loadUser(id: userId)
        }
    }
}
```

### When to Use @Observable

Use `@Observable` for:

- ViewModels
- Repositories
- Services (NetworkService, AuthService, etc.)
- Any class that holds state

**Don't use** `@Observable` for:

- Value types (structs) - Use `@State` instead
- Single-use state - Use `@State` directly

## State Management Patterns

### @State - Local UI State

```swift
struct CounterView: View {
    @State private var count = 0

    var body: some View {
        VStack {
            Text("Count: \(count)")
            Button("Increment") {
                count += 1
            }
        }
    }
}
```

### @Bindable - Two-way Binding

```swift
struct EditUserView: View {
    @Bindable var viewModel: UserProfileViewModel

    var body: some View {
        Form {
            // Bindable allows $ syntax for Observable objects
            TextField("Name", text: $viewModel.user.name)
        }
    }
}
```

### @Environment - Dependency Injection

```swift
// Define environment key
@Observable
class AuthenticationManager {
    var user: User?
    var isAuthenticated: Bool { user != nil }
}

// Inject into environment
@main
struct MyApp: App {
    @State private var authManager = AuthenticationManager()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(authManager)
        }
    }
}

// Access from any view
struct ProfileView: View {
    @Environment(AuthenticationManager.self) private var authManager

    var body: some View {
        if let user = authManager.user {
            Text("Hello, \(user.name)")
        }
    }
}
```

### @AppStorage - UserDefaults Persistence

```swift
struct SettingsView: View {
    @AppStorage("hasSeenOnboarding") private var hasSeenOnboarding = false
    @AppStorage("preferredTheme") private var preferredTheme = "auto"

    var body: some View {
        Form {
            Toggle("Has Seen Onboarding", isOn: $hasSeenOnboarding)
            Picker("Theme", selection: $preferredTheme) {
                Text("Auto").tag("auto")
                Text("Light").tag("light")
                Text("Dark").tag("dark")
            }
        }
    }
}
```

## NavigationStack Patterns

### Basic Navigation

```swift
struct ContentView: View {
    var body: some View {
        NavigationStack {
            List {
                NavigationLink("Profile") {
                    ProfileView()
                }
                NavigationLink("Settings") {
                    SettingsView()
                }
            }
            .navigationTitle("Home")
        }
    }
}
```

### Programmatic Navigation

```swift
struct HomeView: View {
    @State private var navigationPath = NavigationPath()

    var body: some View {
        NavigationStack(path: $navigationPath) {
            VStack {
                Button("Go to Profile") {
                    navigationPath.append("profile")
                }
                Button("Go to Settings") {
                    navigationPath.append("settings")
                }
            }
            .navigationDestination(for: String.self) { destination in
                switch destination {
                case "profile":
                    ProfileView()
                case "settings":
                    SettingsView()
                default:
                    Text("Unknown destination")
                }
            }
        }
    }
}
```

### Type-Safe Navigation

```swift
enum Destination: Hashable {
    case profile(userId: String)
    case settings
    case post(postId: String)
}

struct AppView: View {
    @State private var navigationPath = NavigationPath()

    var body: some View {
        NavigationStack(path: $navigationPath) {
            List {
                Button("View Profile") {
                    navigationPath.append(Destination.profile(userId: "123"))
                }
                Button("Settings") {
                    navigationPath.append(Destination.settings)
                }
            }
            .navigationDestination(for: Destination.self) { destination in
                switch destination {
                case .profile(let userId):
                    ProfileView(userId: userId)
                case .settings:
                    SettingsView()
                case .post(let postId):
                    PostDetailView(postId: postId)
                }
            }
        }
    }
}
```

### Navigation Coordinator Pattern

```swift
@Observable
class NavigationCoordinator {
    var path = NavigationPath()

    func push(_ destination: Destination) {
        path.append(destination)
    }

    func pop() {
        if !path.isEmpty {
            path.removeLast()
        }
    }

    func popToRoot() {
        path = NavigationPath()
    }
}

@main
struct MyApp: App {
    @State private var coordinator = NavigationCoordinator()

    var body: some Scene {
        WindowGroup {
            NavigationStack(path: $coordinator.path) {
                HomeView()
                    .navigationDestination(for: Destination.self) { destination in
                        destinationView(for: destination)
                    }
            }
            .environment(coordinator)
        }
    }

    @ViewBuilder
    private func destinationView(for destination: Destination) -> some View {
        switch destination {
        case .profile(let userId):
            ProfileView(userId: userId)
        case .settings:
            SettingsView()
        case .post(let postId):
            PostDetailView(postId: postId)
        }
    }
}

// In any view
struct SomeView: View {
    @Environment(NavigationCoordinator.self) private var coordinator

    var body: some View {
        Button("Go to Profile") {
            coordinator.push(.profile(userId: "123"))
        }
    }
}
```

## Sheet & Fullscreen Cover Patterns

### Simple Sheet

```swift
struct ContentView: View {
    @State private var showingSheet = false

    var body: some View {
        Button("Show Sheet") {
            showingSheet = true
        }
        .sheet(isPresented: $showingSheet) {
            SheetView()
        }
    }
}
```

### Sheet with Item

```swift
struct ContentView: View {
    @State private var selectedUser: User?

    var body: some View {
        List(users) { user in
            Button(user.name) {
                selectedUser = user
            }
        }
        .sheet(item: $selectedUser) { user in
            UserDetailView(user: user)
        }
    }
}
```

### Sheet with Custom Detents

```swift
struct ContentView: View {
    @State private var showingSheet = false

    var body: some View {
        Button("Show Sheet") {
            showingSheet = true
        }
        .sheet(isPresented: $showingSheet) {
            SheetView()
                .presentationDetents([.medium, .large])
                .presentationDragIndicator(.visible)
        }
    }
}
```

## Reusable View Patterns

### Loading View

```swift
struct LoadingView: View {
    var message: String = "Loading..."

    var body: some View {
        VStack(spacing: 16) {
            ProgressView()
            Text(message)
                .foregroundColor(.secondary)
        }
    }
}
```

### Error View

```swift
struct ErrorView: View {
    let message: String
    let retry: () -> Void

    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: "exclamationmark.triangle")
                .font(.system(size: 60))
                .foregroundColor(.red)

            Text("Error")
                .font(.title)
                .bold()

            Text(message)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)

            Button("Retry") {
                retry()
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
}
```

### Empty State View

```swift
struct EmptyStateView: View {
    let icon: String
    let title: String
    let message: String
    let actionTitle: String?
    let action: (() -> Void)?

    init(
        icon: String,
        title: String,
        message: String,
        actionTitle: String? = nil,
        action: (() -> Void)? = nil
    ) {
        self.icon = icon
        self.title = title
        self.message = message
        self.actionTitle = actionTitle
        self.action = action
    }

    var body: some View {
        VStack(spacing: 16) {
            Image(systemName: icon)
                .font(.system(size: 60))
                .foregroundColor(.gray)

            Text(title)
                .font(.title2)
                .bold()

            Text(message)
                .foregroundColor(.secondary)
                .multilineTextAlignment(.center)

            if let actionTitle = actionTitle, let action = action {
                Button(actionTitle) {
                    action()
                }
                .buttonStyle(.borderedProminent)
            }
        }
        .padding()
    }
}

// Usage
EmptyStateView(
    icon: "tray",
    title: "No Posts Yet",
    message: "Create your first post to get started",
    actionTitle: "Create Post"
) {
    // Create post action
}
```

### Async Content View

```swift
enum LoadingState<Value> {
    case idle
    case loading
    case loaded(Value)
    case failed(Error)
}

struct AsyncContentView<Value, Content: View>: View {
    let loadingState: LoadingState<Value>
    let content: (Value) -> Content
    let retry: () -> Void

    var body: some View {
        switch loadingState {
        case .idle:
            Color.clear
        case .loading:
            LoadingView()
        case .loaded(let value):
            content(value)
        case .failed(let error):
            ErrorView(message: error.localizedDescription, retry: retry)
        }
    }
}

// Usage
struct PostListView: View {
    @State private var viewModel = PostListViewModel()

    var body: some View {
        AsyncContentView(
            loadingState: viewModel.loadingState,
            content: { posts in
                List(posts) { post in
                    PostRow(post: post)
                }
            },
            retry: {
                viewModel.loadPosts()
            }
        )
    }
}
```

## List Patterns

### Basic List

```swift
struct PostListView: View {
    let posts: [Post]

    var body: some View {
        List(posts) { post in
            PostRow(post: post)
        }
    }
}
```

### List with Selection

```swift
struct PostListView: View {
    let posts: [Post]
    @State private var selectedPost: Post?

    var body: some View {
        List(posts, selection: $selectedPost) { post in
            PostRow(post: post)
                .tag(post)
        }
    }
}
```

### List with Swipe Actions

```swift
struct PostListView: View {
    @State private var posts: [Post]

    var body: some View {
        List {
            ForEach(posts) { post in
                PostRow(post: post)
                    .swipeActions(edge: .trailing, allowsFullSwipe: false) {
                        Button(role: .destructive) {
                            deletePost(post)
                        } label: {
                            Label("Delete", systemImage: "trash")
                        }

                        Button {
                            sharePost(post)
                        } label: {
                            Label("Share", systemImage: "square.and.arrow.up")
                        }
                        .tint(.blue)
                    }
            }
        }
    }

    private func deletePost(_ post: Post) {
        posts.removeAll { $0.id == post.id }
    }

    private func sharePost(_ post: Post) {
        // Share logic
    }
}
```

### Pull to Refresh

```swift
struct PostListView: View {
    @State private var viewModel = PostListViewModel()

    var body: some View {
        List(viewModel.posts) { post in
            PostRow(post: post)
        }
        .refreshable {
            await viewModel.refresh()
        }
    }
}
```

### Searchable List

```swift
struct PostListView: View {
    @State private var searchText = ""
    let posts: [Post]

    var filteredPosts: [Post] {
        if searchText.isEmpty {
            return posts
        } else {
            return posts.filter { post in
                post.title.localizedCaseInsensitiveContains(searchText) ||
                post.content.localizedCaseInsensitiveContains(searchText)
            }
        }
    }

    var body: some View {
        List(filteredPosts) { post in
            PostRow(post: post)
        }
        .searchable(text: $searchText, prompt: "Search posts")
    }
}
```

## Form Patterns

### Basic Form

```swift
struct SettingsView: View {
    @AppStorage("username") private var username = ""
    @AppStorage("notifications") private var notifications = true
    @AppStorage("theme") private var theme = "auto"

    var body: some View {
        Form {
            Section("Account") {
                TextField("Username", text: $username)
            }

            Section("Preferences") {
                Toggle("Notifications", isOn: $notifications)

                Picker("Theme", selection: $theme) {
                    Text("Auto").tag("auto")
                    Text("Light").tag("light")
                    Text("Dark").tag("dark")
                }
            }

            Section {
                Button("Sign Out") {
                    // Sign out
                }
                .foregroundColor(.red)
            }
        }
    }
}
```

### Form Validation

```swift
struct SignUpView: View {
    @State private var email = ""
    @State private var password = ""
    @State private var confirmPassword = ""

    var isValid: Bool {
        !email.isEmpty &&
        email.contains("@") &&
        password.count >= 8 &&
        password == confirmPassword
    }

    var body: some View {
        Form {
            TextField("Email", text: $email)
                .textContentType(.emailAddress)
                .autocapitalization(.none)

            SecureField("Password", text: $password)
                .textContentType(.newPassword)

            SecureField("Confirm Password", text: $confirmPassword)
                .textContentType(.newPassword)

            if !password.isEmpty && password.count < 8 {
                Text("Password must be at least 8 characters")
                    .foregroundColor(.red)
                    .font(.caption)
            }

            if !confirmPassword.isEmpty && password != confirmPassword {
                Text("Passwords don't match")
                    .foregroundColor(.red)
                    .font(.caption)
            }

            Button("Sign Up") {
                // Sign up
            }
            .disabled(!isValid)
        }
    }
}
```

## ViewModifier Patterns

### Reusable Card Style

```swift
struct CardModifier: ViewModifier {
    func body(content: Content) -> some View {
        content
            .padding()
            .background(Color(.secondarySystemBackground))
            .cornerRadius(12)
            .shadow(radius: 2)
    }
}

extension View {
    func cardStyle() -> some View {
        modifier(CardModifier())
    }
}

// Usage
Text("Hello")
    .cardStyle()
```

### Loading Overlay

```swift
struct LoadingOverlay: ViewModifier {
    let isLoading: Bool

    func body(content: Content) -> some View {
        content
            .overlay {
                if isLoading {
                    ZStack {
                        Color.black.opacity(0.3)
                        ProgressView()
                            .tint(.white)
                    }
                }
            }
            .allowsHitTesting(!isLoading)
    }
}

extension View {
    func loadingOverlay(_ isLoading: Bool) -> some View {
        modifier(LoadingOverlay(isLoading: isLoading))
    }
}

// Usage
ContentView()
    .loadingOverlay(viewModel.isLoading)
```

## Common Gotchas

### 1. Observable Macro Requires iOS 17+

For iOS 16 and earlier, use `ObservableObject`:

```swift
class ViewModel: ObservableObject {
    @Published var data: [String] = []
}

struct MyView: View {
    @StateObject private var viewModel = ViewModel()
}
```

### 2. State vs Binding

- Use `@State` to own state
- Use `@Binding` to share state

```swift
struct ParentView: View {
    @State private var isOn = false

    var body: some View {
        ChildView(isOn: $isOn)
    }
}

struct ChildView: View {
    @Binding var isOn: Bool

    var body: some View {
        Toggle("Switch", isOn: $isOn)
    }
}
```

### 3. Task Cancellation

Always handle task cancellation:

```swift
.task {
    do {
        let data = try await loadData()
        self.data = data
    } catch is CancellationError {
        // Task was cancelled, don't show error
    } catch {
        errorMessage = error.localizedDescription
    }
}
```

### 4. Avoid Nested Observables

Don't nest @Observable classes:

```swift
// BAD
@Observable
class ParentViewModel {
    var childViewModel = ChildViewModel() // Don't do this
}

// GOOD - Use environment instead
@main
struct MyApp: App {
    @State private var parentVM = ParentViewModel()
    @State private var childVM = ChildViewModel()

    var body: some Scene {
        WindowGroup {
            ContentView()
                .environment(parentVM)
                .environment(childVM)
        }
    }
}
```

## Bottom Line

Use @Observable for iOS 17+. Use NavigationStack for navigation. Use environment for dependency injection. Keep views simple. Extract reusable components. Don't fight SwiftUI's declarative nature.
