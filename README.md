# HTCC Summer Camp 2026 — Digital Register

A phone-friendly attendance register for the Harrow Town CC junior summer cricket camp
(20 July – 14 August 2026). Coaches open a link, tap children in and out, and see a live
count of how many are on site.

---

## What coaches see

- **On site now** — a large live count at the top. This is the number that matters at 3pm.
- **Day picker** — all 20 camp days. Opens on today's date automatically during the camp.
- **One row per child booked that day** — only the children actually booked appear, so the
  Monday list is different from the Tuesday list.
- **Sign in / Sign out** — one tap each, timestamped. Tap again to undo a mistake.
- **📞** — dials the parent's mobile directly.
- **Collected by / Notes** — free text, for who picked the child up and anything worth recording.
- **+ Walk-in** — add a child who turns up without a booking.
- **Export CSV** — downloads that day's register.
- **Sign all out** — end-of-day catch-all.

---

## Deploying to GitHub Pages

1. Create a repo (or add these files to an existing one), e.g. `philanm/htcc-camp-register`.
2. Upload `index.html` and `manifest.json` to the root of the repo.
3. In the repo: **Settings → Pages → Source: Deploy from a branch**, branch `main`, folder `/ (root)`.
4. Wait a minute, then share the URL with the coaches:

   `https://philanm.github.io/htcc-camp-register/`

Coaches can tap **Share → Add to Home Screen** on their phone to use it like an app.

---

## Turning on live sync between coaches (important)

Out of the box the register saves to **the phone it's used on only**. That's fine for a single
coach, but if two coaches both mark children in, they won't see each other's entries.

To sync in real time, paste in your existing Firebase config — the same one the Sunday juniors
register uses. Near the top of the `<script>` block in `index.html`:

```js
const FIREBASE_CONFIG = {
  apiKey: "PASTE_YOUR_API_KEY",
  authDomain: "PASTE.firebaseapp.com",
  databaseURL: "https://PASTE-default-rtdb.europe-west1.firebasedatabase.app",
  ...
};
const DB_ROOT = "summerCamp2026";
```

Replace the `PASTE...` values with your real config. `DB_ROOT` keeps the camp data in its own
branch of the database, so it will **not** interfere with the Sunday register's data.

The indicator in the top-right of the header tells you which mode you're in:

| Indicator | Meaning |
|---|---|
| 🟢 **Synced** | Firebase connected — all coaches see the same register live |
| 🟡 **This device only** | No config — data saves only on this phone |

---

## Submitting the register to Google Sheets

The **📤 Submit register** button sends a timestamped snapshot of that day's register
(every booked child, their status, sign-in/out times, collected-by, notes, and the
coach/lead names) to a new **Attendance Log** tab in the
[Summer Camp Registration 2026 (Responses)](https://docs.google.com/spreadsheets/d/1ny2KWmWMTTIJT9usjGS28L4a2gnfQcpdhiwgD9zY0zs/edit)
sheet. Coaches can submit as many times as they like during the day (e.g. once at
pickup, again if a late correction comes in) — each submission just adds more rows,
nothing is overwritten, so there's always a record to point to if a parent later has
a question about when their child was signed in or out.

Because this site is static (no server), sending data to Sheets needs a small piece of
glue: a **Google Apps Script Web App** bound to the spreadsheet. This is a one-off setup
step, done once by whoever owns the sheet:

1. Open the spreadsheet → **Extensions → Apps Script**.
2. Delete the placeholder code and paste in:

   ```js
   const SHARED_SECRET = "CHANGE_ME"; // must match APPS_SCRIPT_SECRET in index.html
   const LOG_SHEET_NAME = "Attendance Log";

   function doPost(e) {
     try {
       const body = JSON.parse(e.postData.contents);
       if (body.secret !== SHARED_SECRET) {
         return jsonOut({ ok: false, error: "Unauthorized" });
       }
       const ss = SpreadsheetApp.getActiveSpreadsheet();
       let sheet = ss.getSheetByName(LOG_SHEET_NAME);
       if (!sheet) {
         sheet = ss.insertSheet(LOG_SHEET_NAME);
         sheet.appendRow(["Submitted at","Date","Day","Child","Age","Status",
           "Time in","Time out","Collected by","Notes","Coach","Lead coach"]);
         sheet.setFrozenRows(1);
       }
       const submittedAt = body.submittedAt || new Date().toISOString();
       const rows = (body.rows || []).map(r => [
         submittedAt, r.date, r.day, r.child, r.age, r.status,
         r.timeIn, r.timeOut, r.collectedBy, r.notes, r.coach, r.lead
       ]);
       if (rows.length) {
         sheet.getRange(sheet.getLastRow() + 1, 1, rows.length, rows[0].length).setValues(rows);
       }
       return jsonOut({ ok: true, added: rows.length });
     } catch (err) {
       return jsonOut({ ok: false, error: String(err) });
     }
   }

   function jsonOut(obj) {
     return ContentService.createTextOutput(JSON.stringify(obj))
       .setMimeType(ContentService.MimeType.JSON);
   }
   ```

3. Change `CHANGE_ME` to a secret string of your choosing (any random phrase works).
4. **Deploy → New deployment → type: Web app.**
   - Execute as: **Me**
   - Who has access: **Anyone**
   - Click **Deploy**, and authorize the permissions Google asks for (this is your
     own script accessing your own sheet).
5. Copy the **Web app URL** it gives you.
6. In `index.html`, near the top of the `<script>` block, paste it in:

   ```js
   const APPS_SCRIPT_URL = "PASTE_YOUR_APPS_SCRIPT_WEB_APP_URL";
   const APPS_SCRIPT_SECRET = "CHANGE_ME"; // same string as in the script
   ```

7. Commit and push. The Submit button will now write to the sheet.

**Security note:** the secret lives in the page's JavaScript, so it stops the URL being
found and spammed by chance, but it is not a substitute for real authentication —
anyone who reads the page source could see it. That's an acceptable trade-off for an
internal coach tool on a link that isn't advertised publicly, but don't treat it as
protecting sensitive data.

If `APPS_SCRIPT_URL` is left as the placeholder, the Submit button tells the coach
setup isn't finished yet rather than failing silently.

---

## Updating the roster

The children are baked into `index.html` in the `ROSTER` array:

```js
const ROSTER = [{"n":"Arjun Patel","a":7,"p":"Vipul Patel","t":"7877857027","d":[10,11,...]}, ...];
```

- `n` name, `a` age, `p` parent, `t` parent mobile
- `d` = the days they're booked, numbered **0–19** across the camp
  (0 = Mon 20 Jul, 4 = Fri 24 Jul, 5 = Mon 27 Jul … 19 = Fri 14 Aug)

If bookings change, edit that array and commit. Last generated from the registration
sheet with **21 children**.

---

## Safeguarding

The footer carries the standing reminders: minimum two adults on site at all times, children
released only to a named parent or guardian, emergencies to 999. The register is a record of
attendance — it does not replace the club's accident book or incident reporting.
