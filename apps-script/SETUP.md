# Google Sheets sync — one-time setup (~10 min)

This turns a Google Sheet into the shared, faction-only backend for the hit list.
Do these steps once, then send Claude the **Web app URL**.

## 1. Create the Sheet
- Go to <https://sheets.new> (a blank Google Sheet). Name it e.g. **BKM Hit List**.
- Leave it empty — don't add tabs or headers. The script builds a `Targets` tab automatically.

## 2. Add the script
- In the Sheet: **Extensions → Apps Script**.
- Delete whatever's in the editor, then paste the entire contents of [`Code.gs`](./Code.gs).
- Click the **Save** icon (💾).

## 3. Deploy as a Web App
- Top-right: **Deploy → New deployment**.
- Click the ⚙ gear → choose **Web app**.
- Set:
  - **Description:** `BKM hitlist API`
  - **Execute as:** **Me**
  - **Who has access:** **Anyone**   ← required so the app can call it without a Google login
- Click **Deploy**.

## 4. Authorize (one time)
- Google will ask you to authorize. Pick your account.
- You'll see **“Google hasn't verified this app.”** That's normal for a personal script.
  → Click **Advanced** → **Go to BKM Hit List (unsafe)** → **Allow**.
- (You're granting it access to *your* spreadsheet and permission to call the Torn API. No secrets are stored.)

## 5. Copy the URL
- After deploying, copy the **Web app URL** — it ends in **`/exec`**, like:
  `https://script.google.com/macros/s/AKfy..................../exec`
- **Send that URL to Claude.** That's it on your end.

---

### Notes
- **“Anyone” access is safe here:** every action requires a Torn API key that the
  script verifies belongs to faction **56875** before reading or writing. Randoms
  who find the URL can't do anything.
- **If the script is ever updated:** in Apps Script, **Deploy → Manage deployments
  → ✏️ edit → Version: New version → Deploy.** The `/exec` URL stays the same.
- The Sheet stays **private** — don't share/publish it. Access flows only through
  the verified script, so the list is not publicly readable.
