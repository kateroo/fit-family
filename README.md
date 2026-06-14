# Summer Streak Challenge 🏆

A simple family exercise-tracking web app. Each summer, family members log a
daily workout to earn gold stars, build streaks, and climb a shared leaderboard.
The whole point is encouragement, not competition — *you win just by playing.*

## What it does

- **Log your day:** pick your name, answer "did you earn a gold star today?" and
  (optionally) record what you did and for how many minutes, plus a daily message.
- **Leaderboard:** shows everyone's status for today, current streak, and total
  points, with a 🏆 next to whoever's leading.
- **Personal summary:** click a name to see total minutes and a breakdown by
  activity.
- **Edit past days:** click *edit* to open a calendar and fix any day.

## How it's built

A single, self-contained HTML file — no build step, no server of our own.

- **`fit-tracker.html`** — the app the family uses.
- **`fit-tracker-for-Von.html`** — an alternate version that adds daily 🥇/🥈
  medals and a "Yesterday's Winners" banner. Kept for reference; not the one in use.
- **[Tailwind CSS](https://tailwindcss.com/)** (via CDN) for styling.
- **[Firebase](https://firebase.google.com/)** (Firestore + Anonymous Auth) for
  shared data, so everyone sees the same leaderboard in real time.

Just open `fit-tracker.html` in a browser to run it.

## Scoring rules

- **+10 points** for every day you earn a gold star (1+ minute of exercise).
- **Streak bonuses:** +25 points the first time you reach a **5-day**, **15-day**,
  and **25-day** streak. Each milestone is earned **once per person** — breaking
  and restarting a streak does *not* re-award it.
- **Streak flames** appear next to your name: 🔥 at 3 days, scaling up to
  🔥🔥🔥🔥🔥 at 30+.
- **Ranking** is by total points; ties are broken by total minutes exercised.

## Maintenance

### Run a new season (change the year)

Edit the two date constants near the top of the `<script>` in `fit-tracker.html`:

```js
const challengeStartDate = new Date('2026-06-08T00:00:00Z');
const challengeEndDate   = new Date('2026-09-30T23:59:59Z');
```

Also update the date shown in the header (`<p>June 8, 2026 - September 30, 2026</p>`).
The edit-calendar months are derived from these dates automatically.

### Family members

The roster lives in one line in `fit-tracker.html`:

```js
const familyMembers = ["Von", "Bob", "Mom", "Kate", "Jen"];
```

If you change it, also update the allowed names in `firestore.rules` (the
`isFamily` list) and redeploy the rules (below).

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
