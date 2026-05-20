# Daily Exercise Tracker

A mobile-first personal workout tracker built as a single-page web app. No framework, no build step — just HTML, CSS, and JavaScript. Workout data syncs to Firebase Firestore in real time and is cached locally in IndexedDB for offline use.

---

## What it does

### Today
The main workout screen. Shows the current day, a rotating motivational quote, and three live stats — exercises done, total sets, and total volume in lbs. Select your routine from the dropdown, then expand each exercise card to log sets with weight and reps. Tap **Log** to mark a set done. A progress bar tracks completion across the whole session.

### Plan
Browse all four routines and their exercises. Shows the planned sets, reps, and rest time for each exercise, plus the best weight you've ever lifted for it.

### History
A reverse-chronological log of your last 20 sessions. Each entry shows the date, routine, and a breakdown of every exercise logged with set count and best weight.

### Monthly
A calendar view of the current month. Days with logged sessions are highlighted. Shows your current workout streak and monthly totals — sessions, sets, and volume.

### Progress
A 20-week body composition plan with a bar chart showing projected body fat waypoints. Breaks the programme into three phases (Foundation, Progression, Final Push) with guidance on what to expect each phase.

### Milestones
Six milestone cards at 2, 4, 8, 12, 16, and 20 weeks describing expected changes in body composition, strength, and habit formation.

### Strength
Per-exercise strength progression charts. Select a routine filter to narrow the view. Each exercise shows a line chart of max weight lifted over time, plus the total gain or loss since your first session.

---

## Routines

| Routine | Exercises |
|---------|-----------|
| **Push** | DB flat bench press, DB incline bench press, DB flyes, DB pullover, DB overhead tricep extension, Shoulder press, Arnold press, Lateral raises |
| **Pull** | Single-arm DB row, Incline DB row, DB pullovers, Rear delt flyes, DB bicep curls, Hammer curls, DB shrugs |
| **Legs** | Bodyweight squats, Lunges, Glute bridges, Calf raises, Wall sit, Step-ups |
| **Abs** | Flutter kicks, Plank, Crunches, Leg raises, DB Russian twists, Dead bug, Bicycle crunches, Side plank, Bird dog, Superman hold, Mountain climbers |

---

## Installation

No package manager or build step required.

**1. Clone the repo**
```bash
git clone https://github.com/siddharthuppal/daily-exercise-tracker.git
cd daily-exercise-tracker
```

**2. Set up Firebase**

Create a project at [console.firebase.google.com](https://console.firebase.google.com), then:

- **Firestore**: Build → Firestore Database → Create database
- **Auth**: Build → Authentication → Sign-in method → Google → Enable
- **Authorized domain**: Authentication → Settings → Authorized domains → add your domain (e.g. `yourusername.github.io`)

Register a Web app in the project and copy the config into `firebase-config.js`:

```js
window.WORKOUT_TRACKER_FIREBASE_CONFIG = {
  apiKey: '...',
  authDomain: '...',
  projectId: '...',
  storageBucket: '...',
  messagingSenderId: '...',
  appId: '...',
  appUserId: 'your-chosen-id',
};
```

`appUserId` sets the Firestore path: `users/{appUserId}/sessions/{date}`.

**3. Set Firestore security rules**

In Firebase Console → Firestore → Rules:

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

**4. Update the allowed email**

In `index.html`, update the `ALLOWED_EMAIL` constant to your Google account email:

```js
const ALLOWED_EMAIL = 'your-email@gmail.com';
```

**5. Deploy**

Push to GitHub and enable GitHub Pages (Settings → Pages → Deploy from `main` branch). Or open `index.html` directly in a browser for local use — Firebase sign-in requires HTTPS or `localhost`, so use a local server for testing:

```bash
npx serve .
# or
python3 -m http.server 8080
```

---

## Usage

1. Open the app — you'll be prompted to sign in with Google
2. On the **Today** tab, select your routine for the day from the dropdown
3. Expand an exercise card and enter weight + reps for each set, then tap **Log**
4. Completed exercises show a checkmark; the progress bar and stats update live
5. Use **+ Add set** to log extra sets beyond the planned count
6. Your session saves automatically — locally first, then synced to Firestore
7. The sync chip in the banner shows the current state: **Cloud sync active** (green), **Sync pending locally** (amber), or **Local only** (grey)

---

## Offline support

The app uses IndexedDB as a local cache. If Firestore is unreachable, it continues working from local data and syncs when connectivity is restored. The sync chip reflects the current state at all times.

---

## Tech stack

| Layer | Technology |
|-------|-----------|
| UI | Vanilla HTML, CSS, JavaScript (ES modules) |
| Local storage | IndexedDB |
| Cloud sync | Firebase Firestore |
| Auth | Firebase Authentication (Google Sign-In) |
| Hosting | GitHub Pages |
| Fonts | Syne, DM Sans, DM Mono (Google Fonts) |

---

## Security

- Google Sign-In is required to access the app
- Only the email in `ALLOWED_EMAIL` can authenticate — all others are rejected immediately
- Firestore rules enforce the same email restriction server-side, so the database is protected even against direct API calls
- No secrets are stored in the repo — the Firebase `apiKey` is a public project identifier, not a credential
