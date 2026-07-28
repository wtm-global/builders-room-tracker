# The Builder's Room — Speaker & Topic Tracker (self-hosted)

A shared, real-time speaker/topic tracker for recruiting guests for *The Builder's Room*. This version runs on your own GitHub Pages URL with a free Firebase database behind it, so it works independently of Claude.

No coding experience required — just following steps and pasting values. Should take about 15–20 minutes.


---

## Part 1 — Create the database (Firebase)

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and sign in with a Google account.
   - Use a **shared WTM/Technovation Google account** if you have one, rather than someone's personal account — that way more than one organizer can manage this later without depending on one person's login.
2. Click **Add project** (big blue/white tile, or a button near the top of the project list).
   - Enter a name, e.g. `builders-room-tracker`. Firebase will generate a Project ID underneath it automatically — you can leave that as-is.
   - Click **Continue**.
   - You'll be asked about Google Analytics. Toggle it **off** — not needed for this tool. Click **Create project**.
   - Wait for the "Your new project is ready" screen, then click **Continue**.
3. You're now in the project's dashboard. In the **left sidebar**, look for **Firestore** (it may appear under a section called **Databases & Storage**, or under **Build**, depending on when you're reading this — Google reorganizes this menu periodically. The icon looks like a small stack/flame). Click it.
4. Click **Create database** (or **Add database**). As of mid-2026, this opens a multi-step panel:
   - **Step 1 — Select edition:** choose **Standard edition** (the left option). This is the free-tier, general-purpose option and is what this tool needs — you don't need **Enterprise edition**. Click **Next**.
   - **Step 2 — ID, Location, & API type:**
     - **Database ID** — leave this as the default (usually `(default)`) unless you already have a reason to name it something else.
     - **Location** — pick a region physically close to your team (e.g. `nam5` / `us-central` if you're US-based). You generally can't change this later, but for a low-traffic internal tool the exact choice doesn't matter much.
     - You will **not** be asked to choose between "Native mode" and "Datastore mode" here — the Firebase console (unlike the raw Google Cloud console) only ever creates Native-mode databases, since that's the only mode its SDKs support. If this step instead shows you a **Test mode / Production mode** choice, that's the next bullet below, just appearing a step earlier than described here — go ahead and treat it as Step 3.
     - Click **Next**.
   - **Step 3 — Configure (security rules starting mode):** choose **Test mode**. There's usually a plain-language description next to it saying something like *"anyone can read and write your data for 30 days"* — that's expected and fine for now. We'll replace this with real rules in Part 4, before this goes live.
     - If you're offered a "Realtime Updates" toggle, that only applies to Enterprise edition — you can ignore it on Standard.
   - Click **Create** (or **Create database**). Give it a few seconds to provision.
5. Once the database is created, go to the **gear icon** near the top of the left sidebar → **Project settings**.
6. Scroll down to the **Your apps** card. Click the **`</>`** icon (this registers a *web app* — distinct from the iOS/Android icons next to it).
   - Give it a nickname, e.g. `tracker-web`. It doesn't need to match anything else.
   - When asked about **Firebase Hosting**, leave the checkbox **unticked** — we're using GitHub Pages instead, not Firebase's own hosting.
   - Click **Register app**.
7. Firebase will now show a code block with a `firebaseConfig` object, looking like this:
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "builders-room-tracker.firebaseapp.com",
     projectId: "builders-room-tracker",
     storageBucket: "builders-room-tracker.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef"
   };
   ```
   Copy this whole block somewhere safe (a scratch note is fine) — you'll paste it into the tracker file in the next part. Then click **Continue to console**.

---

## Part 2 — Drop your config into the tracker

1. Get a local copy of `index.html` on your computer (from wherever you saved the file provided with these instructions).
2. Open it in any plain text editor — this does **not** mean opening it by double-clicking (that will launch it in a browser instead). Right-click the file and choose **Open with →** Notepad (Windows), TextEdit (Mac — but see note below), VS Code, or similar.
   - **Mac/TextEdit users:** TextEdit defaults to "rich text" mode, which can corrupt code files. Before editing, go to TextEdit's menu → **Format → Make Plain Text** if it isn't already.
   - If you don't have a text editor preference, VS Code (free, from code.visualstudio.com) is a safe, simple choice.
3. Use **Find** (Ctrl+F / Cmd+F) in your editor to search for the text `FIREBASE_CONFIG`. You should land on a block near the top of the `<script type="module">` section that looks like:
   ```js
   const FIREBASE_CONFIG = {
     apiKey: "YOUR_API_KEY",
     authDomain: "YOUR_PROJECT.firebaseapp.com",
     projectId: "YOUR_PROJECT_ID",
     storageBucket: "YOUR_PROJECT.appspot.com",
     messagingSenderId: "YOUR_SENDER_ID",
     appId: "YOUR_APP_ID"
   };
   ```
4. Carefully replace **only the text inside the quotation marks** for each line with the matching value Firebase gave you in Part 1, step 7. Keep the quotation marks and commas exactly as they are — only the `"YOUR_..."` placeholders change.
   - Example: `apiKey: "YOUR_API_KEY",` becomes `apiKey: "AIzaSyD4f...",` (using your real key).
   - Double-check you haven't accidentally deleted a comma or quotation mark while editing — a missing comma here is the single most common cause of the page showing a blank screen.
5. Save the file (Ctrl+S / Cmd+S). Keep the filename exactly `index.html` — GitHub Pages specifically looks for a file with this name to serve as your homepage.

**How to check this worked before going any further:** double-click `index.html` to open it directly in a browser. You should see a "Sign in to open the tracker" screen with no yellow warning banner above it. If you see a banner saying *"Firebase isn't configured yet,"* one of the placeholder values wasn't fully replaced — go back and check each line.

---

## Part 3 — Turn on sign-in (Firebase Authentication)

The tracker requires Google sign-in before it shows anything, and only lets in:
- Anyone with an **@technovation.org** or **@iridescentlearning.org** email, and
- Any other specific email you add to a guest allowlist (for Ambassadors/volunteers using personal accounts)

1. Back in the Firebase console, look in the **left sidebar** for **Authentication** (it may sit under a **Build** heading, or on its own — again, Google shuffles these labels periodically; the icon looks like a person/key). Click it.
2. If this is the first time opening it, click **Get started**.
3. You'll land on a **Sign-in method** tab showing a list of providers (Email/Password, Google, Apple, etc.). Click **Google** in that list (or click **Add new provider** first if it's not shown by default).
4. A panel opens: toggle **Enable** to on.
5. It will ask for a **Project support email** — pick any email associated with the project (your own, or the shared account from Part 1).
6. Click **Save**.
7. Still in Authentication, click the **Settings** tab, then find **Authorized domains**. You'll see `localhost` already listed by default (useful if you ever want to test the file on your own computer before it's live).
   - **Leave this tab open, or come back to it** — you'll add your GitHub Pages address here once you have it, in Part 5. Sign-in will not work on your live GitHub Pages URL until you complete that step.

**Important:** enabling Authentication only controls *who can attempt to sign in* — it doesn't by itself decide who's allowed to see or edit the tracker's data. That real access decision happens in Firestore's security rules, in the next part. Right now, anyone could sign in with any Google account; they just wouldn't get any data back yet because the old Test-mode rules are still active from Part 1.

---

## Part 4 — Lock down access with Firestore rules

Test mode (from Part 1) leaves the database open to anyone who finds the URL, and it auto-expires after 30 days. Replace it with rules that check the signed-in user's email:

1. Go back to **Firestore** in the left sidebar (Part 1, step 3's location).
2. Click the **Rules** tab along the top of the Firestore page.
3. You'll see an editable code box with the existing test-mode rules. **Select all the text inside that box** (Ctrl+A / Cmd+A while your cursor is in the box) and delete it.
4. Paste in the following, exactly as written:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {

       match /speakers/{docId} {
         allow read, write: if isAllowed();
       }

       // Guest allowlist — manage manually in the console (Part 4b below).
       // Locked to the rules engine only; not readable/writable from the app.
       match /allowlist/{emailId} {
         allow read, write: if false;
       }

       function isAllowed() {
         return request.auth != null && (
           request.auth.token.email.matches('.*@(technovation[.]org|iridescentlearning[.]org)$')
           || exists(/databases/$(database)/documents/allowlist/$(request.auth.token.email))
         );
       }
     }
   }
   ```
5. Click **Publish** (top-right of the Rules editor). It should confirm the rules deployed within a few seconds — there's no additional "are you sure" step, but a small success indicator or timestamp update usually confirms it went through.

With these rules, someone can sign in with any Google account, but Firestore will only hand back (or accept) data if their email matches the two domains above or is on the allowlist. Everyone else sees a clear "not authorized" message in the tracker and gets signed out automatically.

### Part 4b — Adding a guest (non-domain) email

For an Ambassador or volunteer signing in with a personal email (Gmail, etc.) rather than a Technovation/Iridescent Learning address:

1. In the left sidebar, go back to **Firestore**, then click the **Data** tab (next to Rules).
2. Click **Start collection**.
3. For **Collection ID**, type exactly `allowlist` (all lowercase, no spaces). Click **Next**.
4. For **Document ID**, don't use the auto-generated one — instead type their **exact sign-in email, in lowercase** (e.g. `jane.doe@gmail.com`). This must match precisely what Google returns when they sign in, so double-check spelling.
5. You'll be prompted to add at least one field before you can save (Firestore requires this, even though the rule above only checks whether the document exists, not what's in it). Add any field — e.g. Field: `allowed`, Type: `boolean`, Value: `true`.
6. Click **Save**.
7. That person can now sign in immediately — no redeploy or waiting period needed.

To remove someone's access later, open the `allowlist` collection in the **Data** tab, click on their email/document, and delete it (usually a trash icon or an option in a "⋮" menu on that document's page).

---

## Part 5 — Put it on GitHub Pages

1. Go to [github.com](https://github.com). If you don't already have an account, click **Sign up** and create one (a shared team account is worth considering here too, for the same reason as Firebase).
2. Click the **+** icon near the top-right → **New repository**.
   - **Repository name:** anything, e.g. `builders-room-tracker`.
   - **Public** vs **Private:** either works for serving the page itself, but a **private** repo needs a paid GitHub plan (Pro/Team/Enterprise) to use Pages. If you're on a free personal account, choose **Public** — the tracker's actual data still lives safely in Firebase, not in this repo, so making the repo public doesn't expose any speaker data.
   - Leave other options (README, .gitignore, license) unchecked — not needed.
   - Click **Create repository**.
3. On the new (empty) repo page, click **uploading an existing file** (a link in the quick-setup instructions), or go to **Add file → Upload files** from the button near the top.
4. Drag your edited `index.html` (from Part 2) into the upload area, or click to browse and select it.
5. Scroll down and click **Commit changes** (the default commit message is fine).
6. Go to the repo's **Settings** tab (top of the repo page, near Code/Issues/Pull requests).
7. In the left sidebar of Settings, click **Pages**.
8. Under **Build and deployment**:
   - **Source:** select **Deploy from a branch**.
   - **Branch:** select **main**, folder **/ (root)**.
   - Click **Save**.
9. GitHub will show a message like *"Your site is live at..."* — this can take anywhere from **30 seconds to a few minutes** the first time. If the page 404s immediately, wait a bit and refresh.
10. Your URL will look like:
    ```
    https://<your-username>.github.io/builders-room-tracker/
    ```
11. **Go back to Firebase now** (Part 3, step 7) and add this domain — just the host part, e.g. `your-username.github.io` (no `https://`, no trailing path) — to **Authentication → Settings → Authorized domains**. Sign-in will fail on the live site until you do this.
12. Reload your GitHub Pages URL and try **Sign in with Google**. If everything above was done correctly, you should land on the tracker itself.
13. Share the URL from step 10 with your team — that's the tracker.

---

---

## Troubleshooting

| What you see | Likely cause | Fix |
|---|---|---|
| Yellow "Firebase isn't configured yet" banner | A `FIREBASE_CONFIG` placeholder wasn't fully replaced | Re-check Part 2, step 4 — every `"YOUR_..."` value must be gone |
| Blank white page, nothing loads | A quotation mark or comma got dropped while editing the config | Re-copy the config block fresh from Firebase and re-paste carefully |
| "Sign-in was cancelled or failed" | Popup blocked by the browser, or clicked away from the Google popup | Allow popups for this site and try again |
| Stuck on sign-in screen after choosing an account | Your GitHub Pages domain isn't in Firebase's Authorized domains list yet | Part 5, step 11 |
| "This account isn't authorized for the tracker" | Signed-in email doesn't match either domain and isn't on the allowlist | Add them via Part 4b, or confirm they used the right email |
| Rules published but still says not authorized | Browser cached the old sign-in state | Sign out, hard-refresh (Ctrl+Shift+R / Cmd+Shift+R), sign in again |
| Page 404s right after enabling Pages | GitHub hasn't finished deploying yet | Wait 1–2 minutes and refresh |

---

## Keeping it updated later

Whenever you want to change the tracker's design or add a feature, edit `index.html` and re-upload it to the repo (or use `git push` if you're comfortable with git). The Firestore data itself lives separately in Firebase, so updating the page won't affect existing entries.

## Costs

Firebase's free "Spark" tier covers this comfortably — this kind of low-traffic internal tool won't come close to the free quota. GitHub Pages is free for public repos.
