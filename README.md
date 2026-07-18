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

## Updating the roster

The children are baked into `index.html` in the `ROSTER` array:

```js
const ROSTER = [{"n":"Arjun Patel","a":7,"p":"Vipul Patel","t":"7877857027","d":[10,11,...]}, ...];
```

- `n` name, `a` age, `p` parent, `t` parent mobile
- `d` = the days they're booked, numbered **0–19** across the camp
  (0 = Mon 20 Jul, 4 = Fri 24 Jul, 5 = Mon 27 Jul … 19 = Fri 14 Aug)

If bookings change, edit that array and commit. Last generated from the registration
sheet with **18 children**.

---

## Safeguarding

The footer carries the standing reminders: minimum two adults on site at all times, children
released only to a named parent or guardian, emergencies to 999. The register is a record of
attendance — it does not replace the club's accident book or incident reporting.
