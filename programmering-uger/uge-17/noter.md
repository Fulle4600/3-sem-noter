# Uge 17 — Firebase Authentication

Denne uge tilføjer vi rigtig brugerautentificering til vores web-applikationer med Firebase Authentication — Googles cloud-platform til at håndtere login uden at du selv skal bygge og sikre en auth-server.

---

## Hvad er Firebase?

Firebase er Googles cloud-platform til app-udvikling. Det tilbyder:
- **Authentication** — login/signup håndtering
- **Firestore** — cloud database
- **Hosting** — gratis static hosting
- **Storage** — fil-opbevaring

Vi fokuserer på **Firebase Authentication** — som håndterer:
- Bruger-oprettelse
- Login/logout
- Token-håndtering (JWT under motorhjelmen)
- Understøtter: Email/Password, Google, Facebook, GitHub, m.fl.

---

## Opsætning

### 1. Opret Firebase projekt

1. Gå til [console.firebase.google.com](https://console.firebase.google.com)
2. "Add project" → giv det et navn
3. Under "Build" → "Authentication" → "Get started"
4. Aktivér: Email/Password og eventuelt Google Sign-in

### 2. Registrer din web-app

I Firebase Console → Project Settings → "Add app" → Web (</>) → kopier config:

```javascript
// firebase-config.js
import { initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";

const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "dit-projekt.firebaseapp.com",
  projectId: "dit-projekt",
  storageBucket: "dit-projekt.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
```

### 3. Installer Firebase SDK

```bash
npm install firebase
```

Eller via CDN i HTML:
```html
<script type="module">
  import { initializeApp } from "https://www.gstatic.com/firebasejs/10.0.0/firebase-app.js";
  import { getAuth } from "https://www.gstatic.com/firebasejs/10.0.0/firebase-auth.js";
</script>
```

---

## Email/Password Authentication

### Opret bruger (Signup)

```javascript
import { createUserWithEmailAndPassword } from "firebase/auth";
import { auth } from "./firebase-config.js";

async function signup(email, password) {
  try {
    const userCredential = await createUserWithEmailAndPassword(auth, email, password);
    const user = userCredential.user;
    console.log("Bruger oprettet:", user.email);
    // user.uid — unikt bruger-ID
    // user.email — brugerens email
  } catch (error) {
    console.error("Fejl:", error.message);
    // error.code: "auth/email-already-in-use", "auth/weak-password", osv.
  }
}
```

### Login (Sign In)

```javascript
import { signInWithEmailAndPassword } from "firebase/auth";

async function login(email, password) {
  try {
    const userCredential = await signInWithEmailAndPassword(auth, email, password);
    const user = userCredential.user;
    console.log("Logget ind som:", user.email);
  } catch (error) {
    if (error.code === "auth/user-not-found") {
      console.error("Bruger findes ikke");
    } else if (error.code === "auth/wrong-password") {
      console.error("Forkert password");
    } else {
      console.error("Login fejl:", error.message);
    }
  }
}
```

### Logout

```javascript
import { signOut } from "firebase/auth";

async function logout() {
  await signOut(auth);
  console.log("Logget ud");
}
```

---

## Lyt på Auth State (Observer Pattern)

Firebase bruger Observer Pattern til at notificere dig når brugerens login-status ændrer sig:

```javascript
import { onAuthStateChanged } from "firebase/auth";

// Kører automatisk når:
// - siden loader (er brugeren allerede logget ind?)
// - brugeren logger ind
// - brugeren logger ud
onAuthStateChanged(auth, (user) => {
  if (user) {
    console.log("Bruger er logget ind:", user.email, user.uid);
    visLoggetIndUI();
  } else {
    console.log("Ingen bruger logget ind");
    visLoginFormular();
  }
});
```

---

## Google Sign-In

```javascript
import { signInWithPopup, GoogleAuthProvider } from "firebase/auth";

const provider = new GoogleAuthProvider();

async function loginMedGoogle() {
  try {
    const result = await signInWithPopup(auth, provider);
    const user = result.user;
    console.log("Google login:", user.displayName, user.email);
  } catch (error) {
    console.error("Google login fejl:", error.message);
  }
}
```

---

## Integration med Vue.js

```javascript
// App.vue
import { ref, onMounted } from "vue";
import { auth } from "./firebase-config.js";
import { onAuthStateChanged, signOut } from "firebase/auth";

export default {
  setup() {
    const user = ref(null);
    const loading = ref(true);

    onMounted(() => {
      onAuthStateChanged(auth, (firebaseUser) => {
        user.value = firebaseUser;
        loading.value = false;
      });
    });

    const logout = async () => {
      await signOut(auth);
    };

    return { user, loading, logout };
  }
};
```

```html
<template>
  <div v-if="loading">Indlæser...</div>
  <div v-else-if="user">
    <p>Hej {{ user.email }}!</p>
    <button @click="logout">Log ud</button>
  </div>
  <div v-else>
    <LoginForm />
  </div>
</template>
```

---

## Hvordan Firebase Auth virker under motorhjelmen

Firebase Authentication er baseret på JWT-tokens:

1. Bruger logger ind → Firebase validerer credentials
2. Firebase returnerer et **ID Token** (JWT) der er gyldigt i 1 time
3. Tokenet gemmes i browserens `localStorage`
4. Ved API-kald sendes tokenet med: `Authorization: Bearer <token>`
5. Din backend kan verificere tokenet med Firebase Admin SDK

**Det smarte:** Du slipper for at bygge og sikre password-hashing, token-generering, og session-håndtering selv — Firebase håndterer det for dig.

---

## Sikkerhedsregler i Firebase

Firebase bruger Security Rules til at styre hvem der kan læse/skrive data:

```javascript
// Firestore Security Rules eksempel
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Bruger kan kun læse/skrive sine egne dokumenter
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    // Alle kan læse produkter, kun admins kan skrive
    match /products/{productId} {
      allow read: if true;
      allow write: if request.auth.token.admin == true;
    }
  }
}
```

---

## Fejlkoder — de vigtigste

| Fejlkode | Årsag |
|---|---|
| `auth/email-already-in-use` | Email er allerede registreret |
| `auth/invalid-email` | Ugyldig email-format |
| `auth/weak-password` | Password for svagt (under 6 tegn) |
| `auth/user-not-found` | Ingen bruger med denne email |
| `auth/wrong-password` | Forkert password |
| `auth/too-many-requests` | For mange mislykkede loginforsøg — konto midlertidigt blokeret |

---

## Opsummering

| Begreb / Funktion                  | Beskrivelse                                                                    |
|------------------------------------|--------------------------------------------------------------------------------|
| Firebase Authentication            | Googles cloud-service til håndtering af login uden at bygge din egen auth-server |
| `initializeApp(config)`            | Initialiserer Firebase med dine projektindstillinger                           |
| `getAuth(app)`                     | Returnerer Auth-instansen der bruges til alle auth-operationer                 |
| `createUserWithEmailAndPassword`   | Opretter en ny bruger med email og password                                    |
| `signInWithEmailAndPassword`       | Logger en eksisterende bruger ind                                              |
| `signOut(auth)`                    | Logger den nuværende bruger ud                                                 |
| `onAuthStateChanged`               | Observer der lytter på login/logout-ændringer — kører automatisk ved sideload |
| `signInWithPopup`                  | Login via popup-vindue (Google, Facebook osv.)                                 |
| `user.uid`                         | Brugerens unikke ID — bruges til at identificere brugeren i databaser          |
| JWT Token                          | Firebase returnerer et ID token (JWT) der er gyldigt i 1 time                 |
| Security Rules                     | Regler i Firebase der styrer hvem der kan læse/skrive data                    |

---

## Opgaver til selvstudium

1. Opret et Firebase-projekt og aktiver Email/Password authentication i Firebase Console
2. Byg en signup-formular i HTML der kalder `createUserWithEmailAndPassword` og viser brugerens email ved succes
3. Tilføj login-funktionalitet med fejlhåndtering — vis brugervenlige beskeder ved fejl som `auth/email-already-in-use` og `auth/wrong-password`
4. Brug `onAuthStateChanged` til at vise/skjule login-formularen baseret på om brugeren er logget ind
5. Tilføj en "Log ud"-knap der kalder `signOut(auth)` og opdaterer UI'en
6. Forklar med egne ord: Hvad er forskellen på `createUserWithEmailAndPassword` og `signInWithEmailAndPassword`? Hvornår bruges den ene vs. den anden?
