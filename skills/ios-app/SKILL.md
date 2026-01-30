# iOS App Building Skill

> Load this skill when building iOS applications, architecting mobile app features, or setting up iOS project infrastructure.

## Core Philosophy

**90% of your app should be borrowed/bought/copied. 10% is your unique value.**

Stop building authentication systems. Stop building payment flows. Stop building analytics dashboards. Use battle-tested solutions and focus on what makes your app different.

## Component Matrix

| Component | Recommended Solution | Build Custom? | Rationale |
|-----------|---------------------|---------------|-----------|
| **Authentication** | FirebaseUI / Firebase Auth | **NEVER** | Auth is security-critical. Use Firebase Auth with FirebaseUI for pre-built sign-in flows. |
| **Payments** | RevenueCat | **NEVER** | Subscriptions are complex. RevenueCat handles receipt validation, cross-platform syncing, webhooks. |
| **Database** | Firestore / Supabase | **RARELY** | Real-time sync, offline support, security rules. Only build custom for exotic needs. |
| **Push Notifications** | Firebase Cloud Messaging | **NEVER** | APNs is complex. FCM handles token management, targeting, analytics. |
| **Analytics** | Firebase Analytics | **NEVER** | Free, unlimited events, integrates with rest of Firebase. |
| **Crash Reporting** | Crashlytics | **NEVER** | Automatic crash collection, symbolication, alerts. |
| **Your Unique Feature** | Custom Implementation | **YES** | This is your 10%. Build what differentiates your app. |

## Sub-Skill Routing Table

Route to these sub-skills based on the task:

| Task Pattern | Route To | Example Triggers |
|--------------|----------|------------------|
| Setting up sign-in, OAuth, account linking | `firebase-auth/SKILL.md` | "add Google sign-in", "implement authentication", "user login" |
| Implementing subscriptions, paywalls, IAP | `revenuecat/SKILL.md` | "add subscription", "paywall", "in-app purchase" |
| Database setup, data modeling, queries | `firestore/SKILL.md` | "save user data", "database structure", "real-time updates" |
| SwiftUI architecture, state management, navigation | `swiftui-patterns/SKILL.md` | "MVVM setup", "navigation flow", "state management" |
| App Store submission, rejection fixes | `app-store/SKILL.md` | "submit to App Store", "rejection", "app review" |

## What to Focus On

These are worth your time and energy:

1. **Your unique feature** - The 10% that makes your app special
2. **UI/UX polish** - Animations, haptics, delightful interactions
3. **Performance optimization** - Smooth scrolling, fast app launch
4. **Edge case handling** - What happens when network fails, user cancels, etc.
5. **Accessibility** - VoiceOver, Dynamic Type, high contrast
6. **Integration glue** - Connecting pre-built services together
7. **User onboarding** - First-time user experience

## What NOT to Do

Stop wasting time on these:

1. **Building auth from scratch** - Use Firebase Auth + FirebaseUI
2. **Rolling your own payment system** - Use RevenueCat
3. **Custom analytics** - Use Firebase Analytics
4. **Custom crash reporting** - Use Crashlytics
5. **Building a backend from scratch** - Use Firestore, Supabase, or Firebase Functions
6. **Reinventing navigation patterns** - Use SwiftUI NavigationStack
7. **Custom networking layer** - URLSession is fine, or use Alamofire
8. **Building your own image cache** - Use Kingfisher or SDWebImage
9. **Custom date/time formatting** - Use Foundation's formatters
10. **Reinventing the wheel** - Search CocoaPods/SPM first

## Quick Start Template

```swift
import SwiftUI
import FirebaseCore
import FirebaseAuth
import FirebaseFirestore

@main
struct YourApp: App {
    init() {
        // Initialize Firebase
        FirebaseApp.configure()
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

## Common Gotchas

### Info.plist Requirements

Always check Info.plist for required keys:

- `CFBundleURLTypes` - For OAuth redirects
- `NSCameraUsageDescription` - For camera access
- `NSPhotoLibraryUsageDescription` - For photo access
- `ITSAppUsesNonExemptEncryption` - Set to NO if not using custom encryption

### Xcode Project Settings

- Enable "Automatically manage signing" unless you have specific needs
- Set minimum iOS version to iOS 16+ (drops iOS 15 and earlier)
- Enable "Strict Concurrency Checking" for Swift 6 compatibility

### Dependencies Management

Use Swift Package Manager (SPM) for everything possible. Avoid CocoaPods unless absolutely necessary.

```swift
dependencies: [
    .package(url: "https://github.com/firebase/firebase-ios-sdk", from: "10.0.0"),
    .package(url: "https://github.com/RevenueCat/purchases-ios", from: "4.0.0")
]
```

## Testing Strategy

- **Unit tests**: Your business logic (the 10% unique feature)
- **UI tests**: Critical user flows only (sign-in, checkout, core feature)
- **Manual testing**: Edge cases, error states, offline behavior
- **Skip**: Testing third-party SDKs (Firebase, RevenueCat already tested)

## Architecture Principles

1. **MVVM with @Observable** - Use Swift 5.9+ Observation framework
2. **Single source of truth** - One place for each piece of state
3. **Unidirectional data flow** - Data flows down, events flow up
4. **Dependency injection** - Pass dependencies explicitly
5. **Protocol-based design** - For testability and flexibility

## When to Load Sub-Skills

- User mentions authentication/login → Load `firebase-auth/SKILL.md`
- User mentions payments/subscriptions → Load `revenuecat/SKILL.md`
- User mentions database/data → Load `firestore/SKILL.md`
- User asks about SwiftUI patterns → Load `swiftui-patterns/SKILL.md`
- User mentions App Store submission → Load `app-store/SKILL.md`
