# PawLedger Firebase Backend (Production-Ready)

This project now includes a complete Firebase backend for:

- Authentication (email/password + Google)
- Firestore data model for users and pets
- Cloud Functions APIs (`registerPet`, `getUserPets`, `verifyPet`)
- Storage uploads for pet images
- Strict Firestore + Storage security rules
- Frontend integration layer (`useAuth`, `usePets`, services, and Firebase modules)

## Project Structure

```txt
src/
  firebase/
    config.ts
    auth.ts
    firestore.ts
    storage.ts
  services/
    authService.ts
    petService.ts
  hooks/
    useAuth.ts
    usePets.ts

functions/
  src/
    index.ts
    petController.ts
  index.ts
  petController.ts
```

## 1) Firebase Setup Steps

1. Create a Firebase project in the Firebase Console.
2. Enable **Authentication** providers:
   - Email/Password
   - Google (optional but implemented)
3. Create Firestore database in production mode.
4. Enable Firebase Storage.
5. Install Firebase CLI:
   - `npm i -g firebase-tools`
6. Login and link project:
   - `firebase login`
   - `firebase use --add`
7. Copy `.env.example` to `.env` and fill real values:
   - `VITE_FIREBASE_API_KEY`
   - `VITE_FIREBASE_AUTH_DOMAIN`
   - `VITE_FIREBASE_PROJECT_ID`
   - `VITE_FIREBASE_STORAGE_BUCKET`
   - `VITE_FIREBASE_MESSAGING_SENDER_ID`
   - `VITE_FIREBASE_APP_ID`
   - `VITE_FIREBASE_FUNCTIONS_REGION` (optional)
8. Update `.firebaserc` default project id.

## 2) Data Schemas

### users

```json
{
  "uid": "string",
  "name": "string",
  "email": "string",
  "createdAt": "timestamp"
}
```

### pets

```json
{
  "petId": "string (SHA-256)",
  "ownerId": "string (uid)",
  "petName": "string",
  "breed": "string",
  "age": "number",
  "imageUrl": "string|null",
  "serverPetFingerprint": "string (server-derived SHA-256)",
  "createdAt": "timestamp"
}
```

## 3) Security Rules

Rules are provided in:

- `firestore.rules`
- `storage.rules`

Highlights:

- Only authenticated users can access data.
- Users can only read/write pets where `ownerId == request.auth.uid`.
- Storage writes only allowed under `petImages/{uid}/{petId}/{filename}` for owner.
- Storage upload size and content type restrictions applied.

## 4) Cloud Functions APIs

Implemented callable functions:

1. `registerPet`
   - Input: `petId`, `petName`, `breed`, `age`, `imageUrl?`
   - Validates auth + payload + SHA-256 format
   - Enforces `petId` uniqueness
   - Stores secure server fingerprint and timestamp

2. `getUserPets`
   - Requires auth
   - Returns all pets for the logged-in user ordered by newest first

3. `verifyPet`
   - Input: `petId`
   - Returns pet + owner profile info

## 5) Frontend Integration

### Auth

- `src/hooks/useAuth.ts`
- `src/services/authService.ts`
- `src/firebase/auth.ts`

Integrated into:

- `src/pages/SignupPage.tsx`
- `src/pages/LoginPage.tsx`
- `src/pages/Dashboard.tsx` (logout)

### Pets

- `src/hooks/usePets.ts`
- `src/services/petService.ts`
- `src/firebase/storage.ts`

Integrated into:

- `src/components/PetRegistration.tsx` (register pet + image upload)
- `src/pages/Dashboard.tsx` (load real user pets)
- `src/pages/VerifiedDashboard.tsx` (verify by `petId`)

## 6) Deployment Steps

From project root:

1. Install frontend deps:
   - `npm install`
2. Install functions deps:
   - `cd functions && npm install`
3. Build functions:
   - `npm run build`
4. Deploy rules + indexes + functions:
   - `firebase deploy --only firestore:rules,firestore:indexes,storage,functions`

## 7) Run Locally

In terminal 1 (root):

- `npm install`
- `npm run dev`

In terminal 2 (`functions/`):

- `npm install`
- `npm run build`

Optional emulator run:

- `firebase emulators:start --only auth,firestore,functions,storage`

---

## Production Notes

- `petId` is validated on server and uniqueness-enforced in Firestore.
- Server also generates `serverPetFingerprint` so the backend does not trust client-only identifiers.
- All write operations use authenticated context and server timestamps.
- Keep Firebase API keys in `.env` (never hardcode secrets).
