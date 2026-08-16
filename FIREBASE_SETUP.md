# Firebase setup for FRC115Software.org

1. Create a Firebase project, add a **Web app**, then paste its configuration into `firebase-config.js`.
2. In **Authentication → Sign-in method**, enable Email/Password and Google.
3. Create a Cloud Firestore database in production mode.
4. Deploy these Firestore rules. They let users read the leaderboard but only change their own profile. A user may only add XP by completing new arc indexes; mentor-reviewed missions should be awarded from a trusted server or Firebase Cloud Function.

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null && request.auth.uid == userId;
      allow update: if request.auth != null && request.auth.uid == userId
        && request.resource.data.uid == resource.data.uid
        && request.resource.data.points >= resource.data.points;
      allow delete: if false;
    }
  }
}
```

5. Serve the folder from Firebase Hosting, GitHub Pages, Netlify, or another HTTPS host. Firebase Authentication does not work reliably when opening `index.html` directly from disk; add the deployed domain under **Authentication → Settings → Authorized domains**.

## Data shape

Each signed-in user has `users/{uid}`:

```text
{ uid, displayName, points, completed: [0, 3], streak, lastActive, createdAt }
```

The browser awards arc XP for the learning prototype. For a competition-grade leaderboard, move point-awarding into a Cloud Function that validates submissions or mentor approvals before writing points.
