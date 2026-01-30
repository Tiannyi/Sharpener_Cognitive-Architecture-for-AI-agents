---
description: Firestore data models, CRUD, real-time listeners, security rules, repository pattern in iOS.
---

# Firestore Sub-Skill

> Load this skill when implementing database functionality, data persistence, real-time sync, or cloud storage in iOS apps.

## Core Principle

**Use Firestore for 90% of apps.** Real-time sync, offline support, and security rules out of the box. Only build custom backend if you have exotic needs (graph databases, heavy computation, sub-millisecond latency).

## Why Firestore?

| Feature | Benefit |
|---------|---------|
| **Real-time sync** | Data updates across devices instantly |
| **Offline support** | App works without internet, syncs when back online |
| **Security rules** | Server-side validation without backend code |
| **Auto-scaling** | Handles 1 user or 1 million users |
| **Free tier** | 1GB storage, 50K reads/day, 20K writes/day |
| **No server maintenance** | Zero DevOps, just write rules |

## Setup Checklist

### 1. Firebase Console Setup

1. Enable Firestore in Firebase Console
2. Choose region (us-central1, europe-west1, etc.)
3. Start in production mode (we'll add rules later)

### 2. Swift Package Dependencies

```swift
dependencies: [
    .package(url: "https://github.com/firebase/firebase-ios-sdk", from: "10.0.0")
]

// In target dependencies:
.product(name: "FirebaseFirestore", package: "firebase-ios-sdk"),
.product(name: "FirebaseFirestoreSwift", package: "firebase-ios-sdk")
```

### 3. Enable Offline Persistence

```swift
import FirebaseCore
import FirebaseFirestore

@main
struct YourApp: App {
    init() {
        FirebaseApp.configure()

        // Enable offline persistence
        let settings = FirestoreSettings()
        settings.isPersistenceEnabled = true
        settings.cacheSizeBytes = FirestoreCacheSizeUnlimited
        Firestore.firestore().settings = settings
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

## Data Modeling Patterns

### Document Structure

Firestore is a NoSQL database with collections and documents:

```
users (collection)
  └─ userId (document)
      ├─ name: "John Doe"
      ├─ email: "john@example.com"
      └─ createdAt: Timestamp

  posts (subcollection under user)
    └─ postId (document)
        ├─ title: "My Post"
        ├─ content: "..."
        └─ publishedAt: Timestamp
```

### Model Types

```swift
import FirebaseFirestore
import FirebaseFirestoreSwift

struct User: Codable, Identifiable {
    @DocumentID var id: String?
    var name: String
    var email: String
    var photoURL: String?
    var createdAt: Date
    var isPremium: Bool

    // Computed property example
    var displayName: String {
        name.isEmpty ? "Anonymous" : name
    }
}

struct Post: Codable, Identifiable {
    @DocumentID var id: String?
    var authorId: String
    var title: String
    var content: String
    var publishedAt: Date
    var likes: Int
    var tags: [String]

    // Reference to subcollection
    // Don't store as property, query when needed
}

struct Comment: Codable, Identifiable {
    @DocumentID var id: String?
    var postId: String
    var authorId: String
    var content: String
    var createdAt: Date
}
```

### Repository Pattern

```swift
import FirebaseFirestore
import FirebaseFirestoreSwift

@Observable
class UserRepository {
    private let db = Firestore.firestore()
    private let collection = "users"

    var currentUser: User?

    // MARK: - Create

    func createUser(_ user: User) async throws {
        guard let userId = user.id else {
            throw NSError(domain: "UserRepository", code: -1,
                         userInfo: [NSLocalizedDescriptionKey: "User ID is required"])
        }

        try db.collection(collection)
            .document(userId)
            .setData(from: user)
    }

    // MARK: - Read

    func fetchUser(id: String) async throws -> User {
        let document = try await db.collection(collection)
            .document(id)
            .getDocument()

        guard let user = try? document.data(as: User.self) else {
            throw NSError(domain: "UserRepository", code: -1,
                         userInfo: [NSLocalizedDescriptionKey: "User not found"])
        }

        return user
    }

    // MARK: - Update

    func updateUser(_ user: User) async throws {
        guard let userId = user.id else {
            throw NSError(domain: "UserRepository", code: -1,
                         userInfo: [NSLocalizedDescriptionKey: "User ID is required"])
        }

        try db.collection(collection)
            .document(userId)
            .setData(from: user, merge: true)
    }

    func updateUserField(id: String, field: String, value: Any) async throws {
        try await db.collection(collection)
            .document(id)
            .updateData([field: value])
    }

    // MARK: - Delete

    func deleteUser(id: String) async throws {
        try await db.collection(collection)
            .document(id)
            .delete()
    }

    // MARK: - Real-time Listener

    func listenToUser(id: String) -> AsyncStream<User?> {
        AsyncStream { continuation in
            let listener = db.collection(collection)
                .document(id)
                .addSnapshotListener { snapshot, error in
                    if let error = error {
                        print("Error listening to user: \(error)")
                        continuation.yield(nil)
                        return
                    }

                    guard let snapshot = snapshot else {
                        continuation.yield(nil)
                        return
                    }

                    let user = try? snapshot.data(as: User.self)
                    continuation.yield(user)
                }

            continuation.onTermination = { _ in
                listener.remove()
            }
        }
    }
}
```

### Query Examples

```swift
@Observable
class PostRepository {
    private let db = Firestore.firestore()

    // Get all posts
    func fetchAllPosts() async throws -> [Post] {
        let snapshot = try await db.collection("posts")
            .order(by: "publishedAt", descending: true)
            .getDocuments()

        return snapshot.documents.compactMap { document in
            try? document.data(as: Post.self)
        }
    }

    // Get posts by author
    func fetchPosts(authorId: String) async throws -> [Post] {
        let snapshot = try await db.collection("posts")
            .whereField("authorId", isEqualTo: authorId)
            .order(by: "publishedAt", descending: true)
            .getDocuments()

        return snapshot.documents.compactMap { document in
            try? document.data(as: Post.self)
        }
    }

    // Get posts with pagination
    func fetchPosts(limit: Int, startAfter: DocumentSnapshot? = nil) async throws -> ([Post], DocumentSnapshot?) {
        var query = db.collection("posts")
            .order(by: "publishedAt", descending: true)
            .limit(to: limit)

        if let startAfter = startAfter {
            query = query.start(afterDocument: startAfter)
        }

        let snapshot = try await query.getDocuments()

        let posts = snapshot.documents.compactMap { document in
            try? document.data(as: Post.self)
        }

        let lastDocument = snapshot.documents.last

        return (posts, lastDocument)
    }

    // Real-time query
    func listenToPosts() -> AsyncStream<[Post]> {
        AsyncStream { continuation in
            let listener = db.collection("posts")
                .order(by: "publishedAt", descending: true)
                .addSnapshotListener { snapshot, error in
                    if let error = error {
                        print("Error listening to posts: \(error)")
                        continuation.yield([])
                        return
                    }

                    guard let snapshot = snapshot else {
                        continuation.yield([])
                        return
                    }

                    let posts = snapshot.documents.compactMap { document in
                        try? document.data(as: Post.self)
                    }

                    continuation.yield(posts)
                }

            continuation.onTermination = { _ in
                listener.remove()
            }
        }
    }
}
```

## Security Rules

### Rules Language

Security rules run on the server and validate all read/write operations.

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Helper functions
    function isAuthenticated() {
      return request.auth != null;
    }

    function isOwner(userId) {
      return isAuthenticated() && request.auth.uid == userId;
    }

    function isPremium() {
      return isAuthenticated() &&
             get(/databases/$(database)/documents/users/$(request.auth.uid)).data.isPremium == true;
    }

    // Users collection
    match /users/{userId} {
      // Anyone can read user profiles
      allow read: if true;

      // Only the user can create/update their own profile
      allow create: if isOwner(userId);
      allow update: if isOwner(userId);

      // Only the user can delete their profile
      allow delete: if isOwner(userId);
    }

    // Posts collection
    match /posts/{postId} {
      // Anyone can read posts
      allow read: if true;

      // Authenticated users can create posts
      allow create: if isAuthenticated() &&
                       request.resource.data.authorId == request.auth.uid;

      // Only author can update their posts
      allow update: if isAuthenticated() &&
                       resource.data.authorId == request.auth.uid;

      // Only author can delete their posts
      allow delete: if isAuthenticated() &&
                       resource.data.authorId == request.auth.uid;

      // Comments subcollection
      match /comments/{commentId} {
        allow read: if true;
        allow create: if isAuthenticated();
        allow update, delete: if isAuthenticated() &&
                                 resource.data.authorId == request.auth.uid;
      }
    }

    // Premium content
    match /premium/{documentId} {
      allow read: if isPremium();
      allow write: if false; // Only writable via backend
    }
  }
}
```

### Common Rules Patterns

**Public read, owner write:**

```javascript
match /items/{itemId} {
  allow read: if true;
  allow write: if request.auth.uid == resource.data.ownerId;
}
```

**Authenticated read/write:**

```javascript
match /items/{itemId} {
  allow read, write: if request.auth != null;
}
```

**Field validation:**

```javascript
match /posts/{postId} {
  allow create: if request.resource.data.keys().hasAll(['title', 'content', 'authorId']) &&
                   request.resource.data.title.size() > 0 &&
                   request.resource.data.content.size() > 0;
}
```

**Rate limiting (with counter document):**

```javascript
match /users/{userId}/dailyActions/{date} {
  allow write: if request.resource.data.count < 100;
}
```

## Offline Persistence

### Enable Offline Mode

Already enabled in app initialization (see Setup section).

### How It Works

1. All reads use local cache first
2. Writes go to local cache immediately
3. Sync happens in background when online
4. Listeners fire with `fromCache: true` for offline data

### Handle Offline State

```swift
import Network

@Observable
class NetworkMonitor {
    var isConnected = true

    private let monitor = NWPathMonitor()
    private let queue = DispatchQueue(label: "NetworkMonitor")

    init() {
        monitor.pathUpdateHandler = { [weak self] path in
            DispatchQueue.main.async {
                self?.isConnected = path.status == .satisfied
            }
        }
        monitor.start(queue: queue)
    }
}

// In your view
struct PostListView: View {
    @Environment(NetworkMonitor.self) private var networkMonitor

    var body: some View {
        VStack {
            if !networkMonitor.isConnected {
                HStack {
                    Image(systemName: "wifi.slash")
                    Text("Offline - changes will sync when connected")
                }
                .padding()
                .background(Color.yellow.opacity(0.3))
            }

            // Your content
        }
    }
}
```

## Real-time Listeners

### Single Document Listener

```swift
struct UserProfileView: View {
    let userId: String
    @State private var user: User?

    var body: some View {
        VStack {
            if let user = user {
                Text(user.name)
            }
        }
        .task {
            let repository = UserRepository()
            for await user in repository.listenToUser(id: userId) {
                self.user = user
            }
        }
    }
}
```

### Collection Listener

```swift
@Observable
class PostListViewModel {
    var posts: [Post] = []
    private var listenerTask: Task<Void, Never>?

    func startListening() {
        listenerTask = Task {
            let repository = PostRepository()
            for await posts in repository.listenToPosts() {
                self.posts = posts
            }
        }
    }

    func stopListening() {
        listenerTask?.cancel()
    }
}
```

## Batch Operations

```swift
func batchUpdate() async throws {
    let db = Firestore.firestore()
    let batch = db.batch()

    // Update multiple documents
    let ref1 = db.collection("posts").document("post1")
    batch.updateData(["likes": FieldValue.increment(Int64(1))], forDocument: ref1)

    let ref2 = db.collection("posts").document("post2")
    batch.updateData(["likes": FieldValue.increment(Int64(1))], forDocument: ref2)

    // Create a document
    let ref3 = db.collection("notifications").document()
    batch.setData(["message": "New likes!", "timestamp": FieldValue.serverTimestamp()], forDocument: ref3)

    // Commit all at once (atomic)
    try await batch.commit()
}
```

## Transactions

```swift
func transferPoints(from fromUserId: String, to toUserId: String, amount: Int) async throws {
    let db = Firestore.firestore()

    try await db.runTransaction { transaction, errorPointer in
        let fromRef = db.collection("users").document(fromUserId)
        let toRef = db.collection("users").document(toUserId)

        // Read documents
        let fromDoc: DocumentSnapshot
        let toDoc: DocumentSnapshot

        do {
            fromDoc = try transaction.getDocument(fromRef)
            toDoc = try transaction.getDocument(toRef)
        } catch let fetchError as NSError {
            errorPointer?.pointee = fetchError
            return nil
        }

        // Validate
        guard let fromPoints = fromDoc.data()?["points"] as? Int,
              fromPoints >= amount else {
            let error = NSError(domain: "TransferError", code: -1,
                               userInfo: [NSLocalizedDescriptionKey: "Insufficient points"])
            errorPointer?.pointee = error
            return nil
        }

        // Write updates
        transaction.updateData(["points": fromPoints - amount], forDocument: fromRef)

        let toPoints = (toDoc.data()?["points"] as? Int) ?? 0
        transaction.updateData(["points": toPoints + amount], forDocument: toRef)

        return nil
    }
}
```

## Common Gotchas

### 1. Array Fields

Use `FieldValue.arrayUnion` and `FieldValue.arrayRemove`:

```swift
// Add to array
try await db.collection("posts")
    .document(postId)
    .updateData([
        "tags": FieldValue.arrayUnion(["swift", "ios"])
    ])

// Remove from array
try await db.collection("posts")
    .document(postId)
    .updateData([
        "tags": FieldValue.arrayRemove(["obsolete"])
    ])
```

### 2. Server Timestamp

Use `FieldValue.serverTimestamp()` for consistent timestamps:

```swift
try await db.collection("posts")
    .document(postId)
    .updateData([
        "updatedAt": FieldValue.serverTimestamp()
    ])
```

### 3. Increment/Decrement

Use `FieldValue.increment()` for atomic counters:

```swift
try await db.collection("posts")
    .document(postId)
    .updateData([
        "likes": FieldValue.increment(Int64(1))
    ])
```

### 4. Compound Queries

Create composite indexes in Firebase Console for complex queries:

```swift
// This requires a composite index on (authorId, publishedAt)
let posts = try await db.collection("posts")
    .whereField("authorId", isEqualTo: userId)
    .order(by: "publishedAt", descending: true)
    .getDocuments()
```

Firebase will prompt you to create the index with a link in the error message.

### 5. Subcollection Queries

You can't query across subcollections. Use a top-level collection instead:

```swift
// BAD: Can't query all comments across all posts
// /posts/{postId}/comments/{commentId}

// GOOD: Top-level comments collection
// /comments/{commentId} with postId field
```

### 6. Delete Field

To delete a field (not document):

```swift
try await db.collection("users")
    .document(userId)
    .updateData([
        "temporaryField": FieldValue.delete()
    ])
```

## Performance Tips

1. **Index everything you query on** - Create composite indexes
2. **Paginate large queries** - Use `limit()` and `startAfter()`
3. **Cache aggressively** - Enable offline persistence
4. **Denormalize data** - Duplicate data to avoid joins
5. **Use subcollections sparingly** - Can't query across them
6. **Batch writes** - Combine multiple writes into one batch
7. **Avoid large documents** - Max 1MB per document

## Data Modeling Guidelines

### Denormalization

**Store user name with posts to avoid extra reads:**

```swift
struct Post: Codable {
    var authorId: String
    var authorName: String  // Denormalized from users collection
    var title: String
}
```

### One-to-Many

**Use subcollections for one-to-many:**

```
users/{userId}
  └─ posts/{postId}
```

**Or use a top-level collection with a field:**

```
posts/{postId}
  └─ authorId: userId
```

### Many-to-Many

**Use array of IDs or separate junction collection:**

```swift
// Array of IDs (good for small lists)
struct User: Codable {
    var followingIds: [String]
}

// Junction collection (better for large lists)
// followers/{userId_followerId}
struct Follower: Codable {
    var userId: String
    var followerId: String
}
```

## Testing

### Emulator

Use Firebase Emulator for local testing:

```bash
# Install
npm install -g firebase-tools

# Initialize
firebase init emulators

# Start emulator
firebase emulators:start
```

Connect to emulator in code:

```swift
#if DEBUG
let settings = Firestore.firestore().settings
settings.host = "localhost:8080"
settings.isPersistenceEnabled = false
settings.isSSLEnabled = false
Firestore.firestore().settings = settings
#endif
```

## Bottom Line

Use Firestore. Don't build your own backend. Don't manage servers. Don't write SQL. Write security rules. Denormalize data. Enable offline persistence. Use real-time listeners for live updates.
