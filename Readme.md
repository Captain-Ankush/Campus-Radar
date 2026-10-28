# 📍 Campus Radar

A real-time location-sharing web app for college campuses. Friends can connect with each other, see each other live on a satellite map of the campus, and get notified when someone is nearby.

**Live demo:** https://captain-ankush.github.io/Campus-Radar/

---

## Features

- **Google Sign-In** using Firebase Authentication
- **Unique registration number** linked to every account (8 digits, enforced with a Firestore transaction so two users can't claim the same one)
- **Friend system:** search a student by registration number, view their profile, send a request, accept or reject it
- **Privacy-first visibility:** only accepted friends can see your location, never all users
- **Live campus map** with satellite imagery and labelled landmarks (Leaflet.js)
- **Nearby alerts:** banner plus browser notification when a friend is within 150 m
- **Friends list:** tap a friend to fly the map to their location
- **Profile:** editable display name
- **Responsive UI:** collapsible sidebar on desktop, slide-in drawer on mobile
- **Android app** (in progress) using Capacitor for background location tracking

## Tech Stack

| Area | Technology |
|------|------------|
| Frontend | HTML, CSS, JavaScript |
| Map | Leaflet.js, Esri World Imagery tiles |
| Auth | Firebase Authentication (Google) |
| Database | Cloud Firestore (real-time `onSnapshot` listeners) |
| Hosting | GitHub Pages |
| Mobile | Capacitor, `@capacitor-community/background-geolocation` |

## Data Model

| Collection | Purpose | Key fields |
|------------|---------|------------|
| `users` | One document per user (ID = auth UID) | `name`, `email`, `photo`, `regNo`, `lat`, `lng`, `updatedAt` |
| `registrations` | Locks a registration number to one user | `uid`, `name`, `createdAt` |
| `friendRequests` | Pending/accepted/rejected requests | `from`, `to`, `status`, `fromName`, `fromPhoto` |
| `friendships` | Mutual friendships (ID = sorted UIDs joined by `_`) | `users` (array of 2 UIDs), `createdAt` |

Accepting a request runs a Firestore **transaction** that creates the friendship and marks the request as accepted in one atomic step.

## Security

Firestore security rules ensure that:

- a user can only write their own `users` document
- `regNo` must be a string of length 8, `lat`/`lng` must be valid numbers
- a friendship can only be read by the two users in it
- a friend request can only be read by its sender or receiver, and only the receiver can accept or reject it
- a registration number can be created once and never edited or deleted

No location history is stored. Each user document only holds the latest position.

## Getting Started

### 1. Clone the repo
```bash
git clone https://github.com/Captain-Ankush/Campus-Radar.git
cd Campus-Radar
```

### 2. Set up Firebase
1. Create a project in the [Firebase Console](https://console.firebase.google.com)
2. Enable **Authentication → Google** and create a **Firestore** database
3. Copy your web app config into `index.html`:
```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.firebasestorage.app",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```
4. Publish the Firestore security rules described above
5. Add your domains (`localhost`, `127.0.0.1`, your GitHub Pages domain) under **Authentication → Settings → Authorized domains**

### 3. Set your campus
Update `CAMPUS_CENTER`, `lpuBounds` and the `campusPlaces` array in `index.html` with your own college's coordinates.

### 4. Run locally
Open the folder with a local server (for example the VS Code **Live Server** extension). Geolocation only works on `https` or `localhost`.

## Android Build (Capacitor)

```bash
npm install
npx cap sync
npx cap open android
```

Then run the app from Android Studio on an emulator or a real device.

> Keep the project **outside** OneDrive/Google Drive folders. Cloud sync locks build files and breaks Gradle builds.

## Known Limitations

- On the web, location updates stop when the browser tab is closed, so friends see your last known position.
- Background tracking needs the native Android build.
- Google Sign-In inside the native app needs `@capacitor-firebase/authentication` plus an Android OAuth client (SHA-1 fingerprint). This is still being set up.
- Push notifications when the app is fully closed are not implemented yet.

## Roadmap

- [ ] Native Google Sign-In for the Android app
- [ ] Background location with a persistent notification
- [ ] "Last seen" timestamps and offline status
- [ ] Ghost mode and time-limited sharing
- [ ] Building outlines on the map
- [ ] Push notifications (Firebase Cloud Messaging)

## Author

**Ankush Singh Rathore**
[GitHub](https://github.com/Captain-Ankush) · [LinkedIn](https://linkedin.com/in/ankush-singh-rathore-512a13388)
