---
description: Load when writing Firebase Cloud Functions, Firestore security rules, or scheduled tasks.
---

# Backend Sub-Skill: Firebase

> **Load when the task involves Cloud Functions, Firestore triggers, or Firebase project configuration.**

## Project Setup

```bash
# Install Firebase CLI
npm install -g firebase-tools

# Initialize project
firebase init
# Select: Functions, Firestore, Storage, Hosting (as needed)
```

## Cloud Functions Patterns

### HTTP Trigger
```typescript
import { onRequest } from "firebase-functions/v2/https";

export const myEndpoint = onRequest(async (req, res) => {
  // Validate request
  if (req.method !== "POST") {
    res.status(405).send("Method not allowed");
    return;
  }
  // Business logic here
  res.json({ result: "success" });
});
```

### Firestore Trigger
```typescript
import { onDocumentCreated } from "firebase-functions/v2/firestore";

export const onUserCreated = onDocumentCreated("users/{userId}", async (event) => {
  const snapshot = event.data;
  if (!snapshot) return;
  const userData = snapshot.data();
  // Send welcome email, create related docs, etc.
});
```

### Scheduled Function
```typescript
import { onSchedule } from "firebase-functions/v2/scheduler";

export const dailyCleanup = onSchedule("every 24 hours", async (event) => {
  // Cleanup logic
});
```

## Firestore Security Rules

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can only read/write their own data
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }

    // Public read, authenticated write
    match /posts/{postId} {
      allow read: if true;
      allow create: if request.auth != null;
      allow update, delete: if request.auth != null
        && resource.data.authorId == request.auth.uid;
    }
  }
}
```

## Common Patterns

- **Fan-out writes:** Use triggers to denormalize data (e.g., update user name in all their posts)
- **Aggregation:** Use triggers to maintain counters (don't query-and-count at read time)
- **Rate limiting:** Use security rules or a Cloud Function middleware
- **Batch operations:** Use `batch` or `runTransaction` for atomic multi-doc writes

## What NOT to Build

- Custom REST endpoints for basic CRUD — use Firestore SDK directly from client
- Custom auth middleware — use Firebase Auth SDK
- Custom file upload handling — use Firebase Storage SDK
