# Firebase Setup

Form submissions are saved to Firebase Firestore. Free tier gives you 1GB storage and 50,000 reads/day - more than enough for an intake form.

## 1. Create a Firebase Project

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project**
3. Give it a name (e.g., "client-intake")
4. Disable Google Analytics if you don't need it (keeps things simpler)
5. Click **Create project**

## 2. Create the Firestore Database

1. In the left sidebar, click **Build > Firestore Database**
2. Click **Create database**
3. Choose a location closest to your clients (e.g., `europe-west1` for South Africa)
4. Select **Start in test mode** (allows read/write for 30 days - see security section below)
5. Click **Create**

## 3. Register a Web App

1. On the project overview page, click the web icon (`</>`) to add a web app
2. Give it a nickname (e.g., "intake-form")
3. You do NOT need Firebase Hosting - leave that unchecked
4. Click **Register app**
5. You'll see a `firebaseConfig` object - copy those values

## 4. Add Your Config to the Form

Open `index.html` and find the `firebaseConfig` object near the top of the `<script>` section:

```javascript
var firebaseConfig = {
  apiKey: "",            // <-- Paste your values here
  authDomain: "",
  projectId: "",
  storageBucket: "",
  messagingSenderId: "",
  appId: ""
};
```

Paste in your values from step 3. It'll look something like:

```javascript
var firebaseConfig = {
  apiKey: "AIzaSyD-abc123...",
  authDomain: "client-intake.firebaseapp.com",
  projectId: "client-intake",
  storageBucket: "client-intake.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123..."
};
```

## 5. Test It

1. Open `index.html` in a browser
2. Fill out the form and submit
3. Go to your Firebase console > Firestore Database
4. You should see a **submissions** collection with a new document containing all the form data

## Where to Find Submissions

In the Firebase console:
- Go to **Firestore Database**
- Click on the **submissions** collection
- Each submission is a document with a random ID
- Click any document to see all the fields and values
- The `submittedAt` field shows when it was submitted

## Security Rules (Important)

Test mode expires after 30 days. Before it does, update your Firestore security rules to only allow writes (not reads) from the public:

1. Go to **Firestore Database > Rules**
2. Replace the rules with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /submissions/{document} {
      allow create: if true;      // Anyone can submit the form
      allow read, update, delete: if false;  // Only you can view/edit/delete via console
    }
  }
}
```

3. Click **Publish**

This means:
- The form can create new submissions (that's all it needs)
- Nobody can read, edit, or delete submissions from the frontend
- You manage submissions through the Firebase console only

## How It Works

- When a client submits the form, all their answers are collected into a JavaScript object
- A `submittedAt` timestamp is added
- The object is saved as a new document in the `submissions` collection in Firestore
- The data is also logged to the browser console as a backup
- If the save fails (network issue, etc.), the success screen shows with an error message telling the client to contact you directly

## Cost

Firebase Spark (free) plan includes:
- 1 GiB Firestore storage
- 50,000 reads / day
- 20,000 writes / day

An intake form submission is 1 write. You'd need 20,000 submissions per day to hit the limit.
