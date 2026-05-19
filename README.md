# daily-exercise-tracker

Mobile-first personal workout tracker built as a single-page web app. Syncs workout data to Firebase Cloud Firestore in real time, with an on-device IndexedDB cache for offline use. Protected by Google Sign-In — only the authorised Google account can access the app or the database.

## Files

- `index.html` — app UI, workout logic, IndexedDB cache, Firestore sync, and auth layer
- `firebase-config.js` — Firebase project config (not committed — see setup below)
- `manifest.json` — PWA metadata

## Firebase setup

1. Create a Firebase project in the [Firebase Console](https://console.firebase.google.com).
2. Add a Web app and copy its `firebaseConfig`.
3. Create a Cloud Firestore database (start in test mode, then lock down rules — see below).
4. Enable Google Sign-In: Authentication → Sign-in method → Google → Enable.
5. Add your hosting domain to: Authentication → Settings → Authorized domains.
6. Fill in `firebase-config.js`:

```js
window.WORKOUT_TRACKER_FIREBASE_CONFIG={
  apiKey:'...',
  authDomain:'...',
  projectId:'...',
  storageBucket:'...',
  messagingSenderId:'...',
  appId:'...',
  appUserId:'your-chosen-id',
};
```

`appUserId` is the Firestore path segment for your data: `users/{appUserId}/sessions/{date}`.

## Security rules

Set these rules in Firestore → Rules before using the app. They require the user to be signed in with a specific Google account — replace the email with your own.

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null
        && request.auth.token.email == 'your-email@gmail.com';
    }
  }
}
```

**Do not use `allow read, write: if true`** — that leaves your data publicly readable and writable by anyone with your project ID.

## Auth

The app shows a Google Sign-In screen on first load. Only the email hardcoded in `ALLOWED_EMAIL` (`index.html`) can sign in — any other Google account is immediately rejected. Access is enforced both client-side (UI gate) and server-side (Firestore rules).

## Sync behaviour

- If Firebase config is blank, the app runs in local-only mode.
- If Firebase is configured, sessions are loaded from Firestore and mirrored into IndexedDB on startup.
- New edits are saved locally first, then synced to Firestore.
- If Firestore is temporarily unavailable, the app continues from local cache and shows the sync state in the UI.
