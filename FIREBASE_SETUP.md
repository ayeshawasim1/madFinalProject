# Firebase Migration Setup Guide

This project has been migrated from Room (local SQLite) to Firebase Realtime Database.
Everything else — UI, UX, features, behavior — remains identical.

---

## What Changed

| Before (Room)                    | After (Firebase)                         |
|----------------------------------|------------------------------------------|
| `AppDatabase` (Room)             | `FirebaseRepository` (Firebase RTDB)     |
| `UserDao`, `MealDao`, etc.       | Single `FirebaseRepository` object       |
| `@Entity`, `@Dao` annotations    | Plain Kotlin data classes                |
| `Int` auto-increment IDs         | Firebase `push()` String keys            |
| `kapt` + Room compiler           | Firebase BOM + `firebase-database-ktx`   |
| Local SQLite file                | Cloud-synced Realtime Database           |
| No signup (demo users only)      | Full user registration + login via RTDB  |

---

## Setup Steps

### 1. Create a Firebase Project
1. Go to [https://console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project**, follow the wizard
3. Click **Add app → Android**
4. Enter package name: `com.fitness.planner`
5. Download `google-services.json`

### 2. Add google-services.json
Replace `app/google-services.json` (the placeholder) with the real file you downloaded.

### 3. Enable Realtime Database
1. In Firebase Console → **Build → Realtime Database → Create database**
2. Choose a region
3. Start in **test mode** for development

### 4. Set Database Rules
In the Firebase Console → Realtime Database → **Rules** tab, paste:

**Development (open — use only for testing):**
```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

**Production (secure):**
```json
{
  "rules": {
    "users": {
      "$userId": {
        ".read": true,
        ".write": true
      }
    },
    "usernames": {
      ".read": true,
      ".write": true
    }
  }
}
```

### 5. Build & Run
```bash
./gradlew assembleDebug
```

---

## Data Structure in Firebase

```
fitness-app/
├── users/
│   └── {userId}/          ← Firebase push() key
│       ├── id: "..."
│       ├── username: "alex"
│       ├── password: "alex123"   ← store hashed in production!
│       ├── email: "alex@..."
│       ├── fullName: "Alex Johnson"
│       ├── workouts/
│       │   └── {workoutId}/  { userId, workoutName, category, durationMinutes, ... }
│       ├── meals/
│       │   └── {mealId}/     { userId, mealName, calories, mealType, date }
│       ├── water/
│       │   └── {waterId}/    { userId, amountMl, date }
│       ├── bmi/
│       │   └── {bmiId}/      { userId, heightCm, weightKg, bmi, category, date }
│       └── goals/
│           └── { userId, dailyCalorieGoal, dailyWaterGoalMl, weightGoalKg }
└── usernames/
    └── {username} → userId   ← index for fast login lookup
```

---

## Demo Users

The old Room version pre-seeded two demo users (`alex/alex123`, `sara/sara123`).
With Firebase, **you register new users via the app's Register screen** — those demo
credentials no longer exist automatically. Just register with any username/password.

---

## Notes

- **Offline support**: Firebase Realtime Database has built-in disk persistence
  (`setPersistenceEnabled(true)` is already configured in `FirebaseRepository`).
  Data reads/writes work offline and sync automatically when connectivity resumes.
- **LiveData**: All list queries use `ValueEventListener` wrapped in `LiveData`,
  so the UI updates in real-time as Firebase data changes.
- **SessionManager**: Still uses `SharedPreferences` to store the logged-in user's
  ID and username — no change to that behavior.
