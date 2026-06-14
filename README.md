# Fit Family Streak Challenge 🍓🏆

A simple family health-tracking web app. Family members log daily check-ins —
exercise, healthy eating, and water — to earn points, build streaks, and climb a
shared leaderboard. The whole point is encouragement, not competition —
*you win just by playing.*

## What it does

- **Three daily check-ins:** pick your name, then answer:
  - ⭐ Did you exercise? (optionally log workout type(s) + minutes, and a daily message)
  - 🥕 Did you eat healthy?
  - 💧 Did you drink enough water? *(for anyone managing fluids, this becomes
    "stayed within your fluid limit" so the healthy choice still earns points)*
- **Leaderboard:** shows each person's check-ins for today, current streak, and
  total points this season, with a 🏆 next to whoever's leading.
- **Personal summary:** click a name to see total minutes and a breakdown by activity.
- **Edit past days:** click *edit* to open a calendar and fix any day.
- **Seasons:** scores reset every 3 months for a fresh start (see below).

## How it's built

A single, self-contained HTML file — no build step, no server of our own.

- **`fit-tracker.html`** — the app the family uses.
- **`fit-tracker-for-Von.html`** — an older alternate version with daily 🥇/🥈
  medals. Kept for reference; not the one in use.
- **[Tailwind CSS](https://tailwindcss.com/)** (via CDN) for styling.
- **[Firebase](https://firebase.google.com/)** (Firestore + Anonymous Auth) for
  shared data, so everyone sees the same leaderboard in real time.
- **`images/`** — header mascots (transparent WebP). One is shown per day —
  **the same one for everyone** — cycling through the set via a fixed shuffled
  order indexed by calendar date. Specific days can be overridden (e.g. Jen's
  birthday on June 18 shows the cake). Source files are kept in `images/originals/`.

Just open `fit-tracker.html` in a browser to run it.

## Scoring rules

A perfect day is **10 points**:

- ⭐ **Exercise** gold star — **5 points**
- 🥕 **Healthy eating** — **2.5 points**
- 💧 **Water** (or staying within a fluid limit) — **2.5 points**
- **Streak bonus:** **+12.5 points** the first time *each season* you reach a
  **5-day**, **15-day**, and **25-day** exercise streak. Earned once per season —
  breaking and restarting a streak does *not* re-award it.
- **Streak flames** appear next to your name: 🔥 at 3 days, scaling up to
  🔥🔥🔥🔥🔥 at 30+.
- **Ranking** is by total points this season; ties broken by total minutes exercised.

## Seasons

The challenge runs year-round in recurring **3-month seasons**. At the start of
each season everyone's score resets to zero (past entries stay in the calendar but
don't carry into the new season's score). Seasons are anchored to **June 15, 2026**
and recur every 3 months (Sep 15, Dec 15, Mar 15, …). The current season's dates
show under the title.

To change the cadence or start, edit these near the top of the `<script>` in
`fit-tracker.html`:

```js
const SEASON_MONTHS = 3;
const seasonAnchor = new Date(2026, 5, 15); // note: month is 0-indexed, so 5 = June
```

## Maintenance

### Family members

The roster lives in one line in `fit-tracker.html`:

```js
const familyMembers = ["Von", "Bob", "Mom", "Kate", "Jen"];
```

If you change it, also update the allowed names in `firestore.rules` (the
`isFamily` list) and redeploy the rules (below). The flipped water question is
currently tied to the name `"Jen"` (search the file for `'Jen'`).

### Adding header mascots

Drop a new image into `images/`, then process it to a transparent WebP and add its
path to the `mascotImages` array in `fit-tracker.html`. (Background removal was done
with a small Pillow flood-fill script; ask Claude to reprocess new ones.) The same
mascot shows for everyone each day, advancing through the list by date.

To show a special mascot on a particular day, add it to the `birthdayMascots` map
(keyed by `"MM-DD"`) right below the array — e.g. `"12-25": "images/mascot-xmas.webp"`.

### Custom workouts

Anyone can pick **Other** in a workout row and type their own activity name — no
code change needed. It shows up in their personal summary with a ❓ icon.

### Firebase / security rules

Data access is controlled by `firestore.rules`. The rules require an (anonymous)
sign-in, restrict writes to the known family names, and validate the data shape.

To change and redeploy the rules (requires the [Firebase CLI](https://firebase.google.com/docs/cli)
and `firebase login`):

```sh
firebase deploy --only firestore:rules
```

Config files: `firebase.json` (points at the rules file) and `.firebaserc`
(sets the default project, `fun-fit-a465a`).

> Note: because everyone shares anonymous sign-in and simply picks a name from a
> dropdown, the rules can't stop one family member from editing another's entry —
> that's by design for a small, trusted group. They *do* stop strangers from
> reading or wiping the data.
