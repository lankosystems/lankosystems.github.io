# Lanko Systems — shop apps

Six single-file HTML apps sharing one Firebase project (`lanko-kairos-cbbcd`) and one
set of design tokens. No build step: each file is opened directly in a browser.

## The naming standard

| Product name                 | Header chip    | File           | Firestore collection | Doc keys  |
| ----------------------------- | -------------- | -------------- | --------------------- | --------- |
| Lanko Aengus Dashboard        | `Aengus 26.4`  | `index.html`   | *(reads only)*        | —         |
| Lanko Kairos Timeclock        | `Kairos 26.11` | `kairos.html`  | `kairos_state`        | `lanko_*` |
| Lanko Minerva Planner         | `Minerva 26.13`| `minerva.html` | `minerva_state`       | `lb_*`    |
| Lanko Kubera Quote Builder    | `Kubera 26.7`  | `kubera.html`  | `mercury_state` ⚠    | `mc_*`    |
| Lanko Iris Shipping & Receiving | `Iris 26.2`  | `iris.html`    | `iris_state`          | —         |
| Lanko Odin Projects           | `Odin 26.6`    | `odin.html`    | `odin_state`          | —         |

The pattern is **`Lanko <Deity> <Function>`**. The deity alone is the short name; the
function says what it does so nobody has to remember which Roman/Greek/Hindu/Norse god
does quoting. Filenames are the lowercase deity, and **never carry a version or a date** —
that is how `Lanko_Mercury_26_7.html` came to look like version 26.7 when 26_7 was
really the 26th of July.

⚠ **Kubera's collection is deliberately still `mercury_state`, and its keys still start
`mc_`.** See "The Mercury rename" below.

## Version rule

Two-digit year, dot, counter. The counter starts at 0 each January and increments once
per released iteration:

```
2026:  26.0  26.1  26.2  …
2027:  27.0  27.1  27.2  …
```

Each app carries its own counter — they are not kept in step. Current: Aengus 26.4,
Kairos 26.11, Minerva 26.13, Kubera 26.7, Iris 26.2, Odin 26.6.

In Aengus the version lives in one place, `APP_VERSION`, and both the `<title>` and the
header chip are built from it. The others still have it written out by hand in a
couple of spots (title, header chip, the "Exit to …" button in Kairos and Minerva, and
the backup-file app stamp in Kubera) — worth collapsing to one constant next time each
is touched.

## Aengus

Aengus is the front door. It opens listeners on the other collections and shows:

- **On the clock** — who is clocked in right now, on what job, since when, plus hours
  logged today. From `kairos_state`.
- **The apps** — plates linking to each app, each carrying its own live vitals.
- **Today's plan** — the published Minerva week, filtered to today. From `minerva_state`.
- **Quote pipeline** — draft/sent/won/lost counts and value. From `mercury_state`.

**Aengus is read-only and must stay that way.** It never writes a document. If the
dashboard could write, a stray click here could clobber a payroll entry or a live quote
with no way to tell which app did it. Anything that changes data belongs in one of the
other five apps.

Links between the apps are relative (`kairos.html`, not a full URL), so the whole set
works unchanged whatever the site ends up being called, and works from a local folder too.

Two things Aengus is careful about, because getting them wrong is worse than useless on a
wall monitor: a panel that **could not be read** says so, rather than showing a
comforting zero; and a Minerva week that was published a while ago is labelled with its
age, so nobody plans a Tuesday off last month's board.

## Back to Aengus

Kairos, Minerva, Kubera and Odin each carry a small pill-shaped link (⌘ Aengus) back
to the dashboard, so nobody has to hit the browser back button or retype the URL. Iris
does not have one — skipped on purpose, not an oversight.

Placement follows whatever survives that app's own re-render, not one fixed rule:

| App | Where | Why there |
| --- | --- | --- |
| Minerva | Wordmark, next to the app name | `topbar-right` gets its innerHTML replaced on every view change |
| Kubera | `topbar-right`, after the sync lamp | Nothing else there gets wholesale-replaced |
| Odin | Header, after Sign out | Same reasoning — `#nav` gets rewritten on tab change, its siblings don't |
| Kairos | Inside the PIN-protected admin panel, next to "Exit to Kairos" | See below |

**Kairos is the odd one out on purpose.** Employees clocking in and out all day don't
need a way off the rack screen, and it's one more thing to accidentally tap on a shop
tablet. The link only appears after the admin PIN, so it's there for Ivan and Travis,
not the crew.

## Minerva: job display and same-job continuity

The week board shows each job as a row in a compact table — job number, name, and hours
remaining — instead of the old chip legend underneath the schedule grid. Same data,
easier to scan at a glance and to compare against remaining hours.

When building a week, Minerva now gives a soft preference to keeping an employee on the
job they most recently clocked into in Kairos (`kairos_state/lanko_entries`, read-only —
Minerva never writes to Kairos's collection). This is a **tie-break only**: it sits below
same-week stickiness (an employee who was on a job Monday gets first claim on it Tuesday,
within the week Minerva is currently computing) and below pins, exclusivity, and job
restrictions. It never overrides priority order or a materials hold, and if the Kairos
read fails or finds no match, scheduling proceeds exactly as before — there is no crash
and no hard dependency on Kairos being reachable.

## Iris

Shipping & Receiving. Shares the same header chrome as the other apps (gradient topbar,
wordmark, version chip) rather than its earlier bespoke banner — one less thing that
looks like a different product when the tablets are lined up on the wall.

## Odin: reminders

Odin now carries a lightweight reminders feature, built for a specific use case: Ivan
and Travis keying in a task from a phone while on the road, and someone actioning it
back at the office.

- A button on the main screen opens a simple "new reminder" screen — a note, a category,
  and an optional project to attach it to.
- Four categories to start: **Procurement, Operations, Scheduling, Other.**
- A reminder attached to a project shows up in two places: the dedicated **Reminders**
  page, and a card on that project's own page. They're the same record — marking it
  done or deleting it from either place updates both.
- Reminders can be marked complete from either the project page or the Reminders page.

Stored under `odin_state`, alongside the rest of Odin's project/procurement data.

## Kairos: Festive Friday

A one-day cosmetic mode for the rack screen — barbed-wire tile borders, a shimmering
banner, a staggered pop-in animation on load, and a synthesized happy chime on
clock-in. Purely decorative: no data model changes, nothing about real clock in/out
behavior is touched.

Gated by `isFestiveFriday()`, which checks the device's local date against a single
hardcoded day. It turns itself on at midnight on that day and off again the next —
no manual toggle, nothing to remember to revert. `PREVIEW_FESTIVE` inside the same
function is a manual override for testing before the date arrives; it must be `false`
in the deployed file or the theme runs every day.

To run it again next year (or for a different one-day theme), update the date check
in `isFestiveFriday()` and the banner text in `renderRack()` — both are in one place,
commented, in `kairos.html`.

## Live quotes in the archive

The archive is history, but not all of it is finished — a quote sent last month may
still be sitting on a customer's desk. Two different actions cover this, and the
difference matters:

- **Quote this again** starts a *new* quote with a *new* number, for re-quoting old work.
- **This one is still live** brings the record into the pipeline *as itself* — same
  number, same revision, same date. Change the number and you break the thread with
  whatever the customer is holding.

Promote one at a time from a document, or tick several in the archive list and move
them together. The archive record is kept either way: it's the evidence of what was
actually sent, and it keeps its link to the original PDF. The two are joined by
`quote.fromArchive`, and the archive list shows the live version instead of the
historical one so a number never appears twice.

Before it moves anything, it warns about the things that bite:

- **A newer revision exists.** Numbers repeat across revisions, so ticking rev 0 when
  rev 2 is on file would drag a superseded document into the pipeline.
- **The lines never reconciled.** Roughly a third of the extracted catalogue has line
  items that don't add up to the printed total — a line that ran across a page break,
  or an option quote where only one option was counted. Those need checking against
  the original before you send anything based on them.
- **Index-only entries** arrive with one empty line to fill in.
- **Unmatched customers** must be picked before the quote will save.

It also nudges the quote counter past any promoted number, so a promoted 26206 can't
collide with the next new quote.

## The Mercury rename

Mercury became Kubera in July 2026. Everything user-facing changed. Everything stored did
not:

- Changed: title, header chip, backup filename, the `app:` stamp inside backups, the
  "Kubera assigns the next one" hint, the archive "Raised here" filter.
- **Unchanged: the `mercury_state` collection and the `mc_*` doc keys.** Renaming them in
  the file would not move the data — it would just point the app at empty documents
  and orphan every quote, pack list and archive row already stored.

New archive rows are stamped `source: 'Kubera'`. Rows already stamped `'Mercury'` are
displayed as Kubera so the column reads consistently. The stored value is left alone.

If the collection is ever renamed properly, it is a copy-then-cut-over, not a
find-and-replace: copy each `mc_*` doc to the new collection, verify, point the app at
it, keep the old collection read-only for a while, then delete.

## Who can sign in, and why

These are static files on a public URL (`lankosystems.github.io`). The Firebase config
inside them is public by design, which means anyone who knows the project id can reach
the database with the REST API without ever loading a page. So any PINs inside the apps
are not security — they run in JavaScript on the visitor's own machine. `firestore.rules`
is the only real boundary, and it works by telling a real account apart from an
anonymous session.

| App | Sign-in | Why |
| --- | --- | --- |
| Kairos | none, anonymous | Shop tablets have no keyboard. Deliberate trade. |
| Minerva | required | Editing the schedule. Kairos still reads it anonymously. |
| Kubera | required | Quote prices, margins, customer contacts. |
| Aengus | optional | Anonymous shows timeclock + planner; quotes need an account. |
| Iris | required | Shipping/receiving records. |
| Odin | required | Project, procurement, and reminder data. |

Aengus is optional on purpose: a wall monitor shouldn't go blank because a session
lapsed, and nobody walking past sees quote values unless someone deliberately signs
in. Its listeners re-attach on sign-in, because a Firestore listener that has already
failed with permission-denied stays dead and the panel would otherwise be stuck on
its error until a reload.

Adding someone: Firebase Console → Authentication → Users → Add user. There are no
roles — any account can read and write everything the rules allow. If that changes,
put an allow-list in the rules rather than in the apps, since the apps are editable
by whoever is looking at them.

**Deploy order, if you ever change this.** Ship the apps that can sign in first,
sign in once to prove it works, then publish the rules. The other way round locks the
office apps out of their own data.

## Still worth tightening

Neither is caused by the rename, but both get more exposed on a public URL.

**1. `kairos_state` is still open to anonymous sessions.** It has to be, so keyboardless
tablets can clock people in. Anyone who works out the project id can read the roster,
jobs and hours. The pricing is what's locked; this is names and times. The proper fix is
App Check, which attests a request came from your real site without adding a login. It
needs SDK changes across every app, so it is not a five-minute job.

**2. Never commit `Lanko_quote_archive.json`.** It is 640 KB of real customer quotes,
names and prices. It went into a public repo once already and had to be deleted along
with the whole repository, because deleting a file leaves it in git history. It only ever
needs to pass through Kubera's Archive tab, which reads it off your disk and writes it
straight to Firestore. It has no reason to be in any repo.
