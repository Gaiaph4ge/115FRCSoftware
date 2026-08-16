# Firebase setup for FRC115Software.org

This site is configured for the Firebase project `frcsoftware-115`.

## Enable authentication (required for sign up)

In the Firebase Console:

1. Open **frcsoftware-115**.
2. Select **Build → Authentication**.
3. Click **Get started** if Firebase asks you to initialize Authentication.
4. Open **Sign-in method**.
5. Enable **Email/Password**, then click **Save**.
6. Enable **Google**, choose a project support email, then click **Save**.
7. Under **Settings → Authorized domains**, add your Firebase Hosting domain, such as `frcsoftware-115.web.app`.

If the website reports that authentication is not enabled, step 5 is the most common missing setting. The code will show a more specific Firebase error in the sign-in dialog.

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

## Deploy the rules and updated site

From this folder, run:

```powershell
firebase deploy --only firestore:rules,hosting
```

The included `firebase.json` points Firestore at `firestore.rules`. Without deploying those rules, authentication can succeed but the profile read/write will fail with **Missing or insufficient permissions**.

## Data shape

Each signed-in user has `users/{uid}`:

```text
{ uid, displayName, points, completed: [0, 3], streak, lastActive, createdAt }
```

The browser awards arc XP for the learning prototype. For a competition-grade leaderboard, move point-awarding into a Cloud Function that validates submissions or mentor approvals before writing points.
