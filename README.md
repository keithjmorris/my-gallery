# My Gallery — setup guide

A personal art gallery web app. Each person signs in and builds their own private gallery: a list of artists, each with artworks, each with one or more photos. Photos open full-screen with pinch-to-zoom.

**Files**

| File | What it is |
|---|---|
| `index.html` | The whole app |
| `firestore.rules` | Security rules for the database, including sharing (paste into Firebase) |
| `storage.rules` | Security rules for photo storage (paste into Firebase) |
| `manifest.json`, `icon-512.png` | Home Screen name and icon |

**Try it first:** until you paste in your Firebase settings, the app runs in *demo mode* with sample artists and nothing is saved. Demo mode includes a sample gallery shared with you by an invented "Sam Carter". Open the folder in VS Code and use the **Live Server** extension (right-click `index.html` → *Open with Live Server*). Double-clicking the file won't work, because browsers block this kind of app from `file://` addresses.

---

## 1. Create the Firebase project

1. Go to <https://console.firebase.google.com> and click **Create a project**. Call it something like `my-gallery`. You can turn Google Analytics off.
2. **Upgrade to the Blaze plan.** Bottom-left of the console, click the plan name → **Upgrade** → Blaze. Storage needs this. You still get the free allowance (5 GB of photos), so a personal gallery will very likely cost nothing.
3. **Set a budget alert** so you're never surprised. In the Google Cloud console go to **Billing → Budgets & alerts → Create budget**, set the amount to, say, £5, and keep the email alerts at 50%, 90% and 100%. A budget only *warns* you; it doesn't stop the app.

## 2. Turn on sign-in

1. In the Firebase console, open **Build → Authentication → Get started**.
2. Under **Sign-in method**, choose **Email/Password**, switch it on, and **Save**.

## 3. Create the database

1. **Build → Firestore Database → Create database**.
2. Location: `europe-west2 (London)` is a good choice. This can't be changed later.
3. Choose **Start in production mode**.
4. Open the **Rules** tab, replace everything with the contents of `firestore.rules`, and click **Publish**.

## 4. Create photo storage

1. **Build → Storage → Get started**.
2. Location: choose **`US-CENTRAL1`** (or another US region). The free 5 GB only applies to US regions.
3. Choose **Start in production mode**.
4. Open the **Rules** tab, replace everything with the contents of `storage.rules`, and click **Publish**.

## 5. Connect the app to Firebase

1. Click the cog ⚙ next to **Project Overview → Project settings**.
2. Under **Your apps**, click the web icon **`</>`**. Give it a nickname (`My Gallery`). Leave “Firebase Hosting” unticked (Vercel does the hosting). Click **Register app**.
3. Firebase shows a `firebaseConfig` block. Copy the six values into the `FIREBASE_CONFIG` section near the top of the `<script>` in `index.html`:

```js
const FIREBASE_CONFIG = {
  apiKey: "AIza…",
  authDomain: "my-gallery-xxxx.firebaseapp.com",
  projectId: "my-gallery-xxxx",
  storageBucket: "my-gallery-xxxx.firebasestorage.app",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

These values are safe to put on GitHub. They identify your project but don't unlock anything. The security rules from steps 3 and 4 are what keep each person's gallery private.

## 6. GitHub and Vercel

1. In VS Code, put these files in a folder and publish it to a new GitHub repository (Source Control panel → **Publish to GitHub**).
2. In Vercel, **Add New → Project**, import the repository, leave the Framework Preset as **Other**, and click **Deploy**. There is no build step.
3. Back in Firebase, go to **Authentication → Settings → Authorized domains → Add domain** and add your Vercel address (for example `my-gallery.vercel.app`).

From now on, every push to GitHub updates the live site automatically.

## 7. On your iPhone

1. Open your Vercel address in **Safari** and tap **Create an account**.
2. Tap the **Share** button → **Add to Home Screen**. The app then opens full-screen with its own icon.
3. Add an artist, then **Add artwork**, then **Add photos**. You can pick several photos of the same artwork at once. The first photo is the main image; tap ★ on another to make it the main one.

---

## How it works

- **Photos** are shrunk on the phone before uploading: the main image to 2,560 pixels on the longest side (about 0.5 MB, still sharp when zoomed) and a small thumbnail for lists. To change these sizes, edit `MAIN_EDGE` and `THUMB_EDGE` near the top of the script.
- **Data layout** (sharing-ready):
  - Database: `users/{yourId}/artists/…` and `users/{yourId}/artworks/…`
  - Storage: `users/{yourId}/artworks/{artworkId}/…jpg`
- **Offline:** artist and artwork details are cached on the phone, so the lists open instantly and still work with a poor signal.
- **In my collection / Seen elsewhere:** every artwork records which it is (the `ownership` field: `mine` or `seen`). Artworks added before this existed show a one-tap question on their page, and the Gallery's **To sort** filter lists them until they're all done.
- **Deleting** an artwork deletes its photos from Storage too. Deleting an artist deletes all of their artworks.

## Sharing

Share your gallery with one person at a time, read-only.

1. Tap the **Share** icon (top right of Artists or Gallery), check your name, and choose what they can see: **Artworks I own**, **Artworks I've seen elsewhere**, or **Everything**.
2. Tap **Create invitation**. A QR code appears. They point their phone's camera at it, or you tap **Send invitation…** to use AirDrop, Messages, WhatsApp and so on.
3. They open it, sign in (or create a free account), and tap **Accept**. Your gallery then appears in their app under **Shared with me**.

**The rules it follows**
- Each invitation works **once**, for **one person**, and expires after 24 hours. If it's forwarded, it won't work for anyone else.
- People you share with can look but never change anything. They see what you add as you add it.
- **Account → People who can see your gallery** lists everyone with access. You can change what each person sees, or remove them, and it takes effect straight away.
- They can't pass access on. They could still take screenshots or show someone their phone, which no app can prevent.
- Artist names and biographies are readable by anyone you share with, even for artists whose artworks they can't see. The app hides those artists, but the details aren't secret from a determined, technically minded viewer.

**Data layout for sharing**
- `invites/{code}`: one-time invitations
- `users/{you}/shares/{theirId}`: who can see your gallery, and what
- `users/{them}/sharedWithMe/{yourId}`: their list of galleries shared with them
- `users/{you}` (the document itself): your display name

### Updating an existing project to the sharing version

1. Replace `index.html`, keeping your own `FIREBASE_CONFIG` block.
2. In Firebase, go to **Firestore Database → Rules**, paste in the new `firestore.rules`, and click **Publish**. `storage.rules` hasn't changed.
3. Push to GitHub.

### Checking the security rules (about 10 minutes)

Use two accounts, for example your own and a second email address, in two different browsers (or one private window).

| Try this | Expected |
|---|---|
| A (you) creates an invitation; B opens it and accepts | B sees A's gallery under "Shared with me" |
| C (a third account) opens the same invitation | "Already used" |
| A changes B to "Seen elsewhere" | B sees only those artworks |
| A removes B | B's view changes to "No longer shared" |
| B, while viewing A's gallery | No Edit, Add or Delete buttons anywhere |
| Wait for an invitation to pass its expiry time, then open it | "This invitation has expired" |

If any step shows "Missing or insufficient permissions", the rules probably weren't published. Check the Rules tab says it was published just now.
