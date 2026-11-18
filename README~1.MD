# EWAIPC MCQEXAM Admin Panel – Firebase Hosting (2026)

## இதில் உள்ளவை
- Responsive dashboard; Exam create/edit/delete/publish; MCQ question CRUD (4 options, correct answer, explanation); student profiles; results, CSV export; Firebase Auth admin login.
- Firebase Hosting + Firestore rules. No custom VPS/backend required.
- Existing Flutter v3 data paths: `papers/{id}`, `papers/{id}/questions/{id}`, `users/{uid}/results/{id}`.

## 1. Firebase web configuration
Firebase Console → `mcq-exam-project` → Project settings → Your apps → Add Web App (if needed) → SDK setup and configuration → Config.
Replace `public/firebase-config.js` with your real `apiKey`, `authDomain`, `projectId`, `storageBucket`, `messagingSenderId`, `appId`. These are web client config values, **not** admin credentials. Do NOT paste any service account private key into web files.

## 2. Enable authentication
Firebase Console → Authentication → Sign-in method → enable Email/Password and/or Google. Add your deployed Firebase Hosting domain to authorized domains if necessary.

## 3. Create FIRST admin securely
1. Firebase Console → Authentication → Users → create or find the intended admin account and copy its UID.
2. Firebase Console → Firestore Database → create collection `admins` → document ID = **the exact Auth UID** → field `active` (boolean) = `true`.
3. **Never** give admin rights by writing `users/{uid}.role` from the browser. The `admins` collection cannot be modified by any client with supplied rules.
4. If admin is disabled, change `active` to false manually in Firebase Console.

## 4. Deploy from Windows PowerShell
Install Node.js LTS and Firebase CLI. In extracted folder:
```
npm install -g firebase-tools
firebase login
firebase use mcq-exam-project
firebase deploy --only hosting,firestore:rules
```
Default URL usually `https://mcq-exam-project.web.app` (verify actual URL shown by CLI). To test locally run `firebase emulators:start --only hosting` (emulator Firestore/Auth configuration may be needed for full testing).

## 5. Student Flutter App compatibility – IMPORTANT
Your previous Flutter v3 reads `papers` using `.orderBy('year', descending: true)` **without filtering `published`**. With secure rules, that query is rejected for students because it could return draft exams. Update the student's query to:
```
FirebaseFirestore.instance.collection('papers')
  .where('published', isEqualTo: true)
  .orderBy('year', descending: true)
  .snapshots()
```
Firebase may prompt you to create a composite index; follow the Firebase error link. The admin panel's draft exams stay invisible to students.
The Flutter app currently stores exam results in `users/{uid}/results`. This panel reads those through collectionGroup('results').

## 6. Student profiles
Firebase Authentication users are not automatically listed by Firestore. After student login, ensure a profile is saved to `users/{uid}` with fields such as `displayName`, `email`, `createdAt` using `set(..., SetOptions(merge:true))`. Do not allow a client to set `role` or `isAdmin`.

## 7. Important production limitations
- Results are **client-scored**, so scores can be falsified by modified clients. For high-stakes exams, move answer checking and result writing to trusted Cloud Functions and hide correct answers until submission. Current Firestore question documents contain `correctIndex`, readable by students taking the exam.
- Firestore rules in this ZIP replace your current rules; **review and merge** if your Firebase project also contains other collections/apps.
- Exam paper delete removes its question subcollection in batches, but does not delete historical results.
- Student profiles list only Firestore `users` documents, not all Firebase Authentication accounts.
- Dashboard counts are based on client reads; for large datasets use paginated queries or server-side aggregation.
- Google login on web uses popup and authorized domain; browsers may block popups.
- The source was generated here; real Firebase credentials and deployment testing are not possible without your Firebase setup.
