# Lanko Systems — shop apps

Four single-file HTML apps sharing one Firebase project (`lanko-kairos-cbbcd`) and one
set of design tokens. No build step: each file is opened directly in a browser.

## The naming standard

| Product name                 | Header chip    | File           | Firestore collection | Doc keys  |
| ---------------------------- | -------------- | -------------- | -------------------- | --------- |
| Lanko Aengus Dashboard       | `Aengus 26.0`  | `index.html`   | *(reads only)*       | —         |
| Lanko Kairos Timeclock       | `Kairos 26.5`  | `kairos.html`  | `kairos_state`       | `lanko_*` |
| Lanko Minerva Planner        | `Minerva 26.1` | `minerva.html` | `minerva_state`      | `lb_*`    |
| Lanko Kubera Quote Builder   | `Kubera 26.1`  | `kubera.html`  | `mercury_state` ⚠    | `mc_*`    |

The pattern is **`Lanko <Deity> <Function>`**. The deity alone is the short name; the
function says what it does so nobody has to remember which Roman/Greek/Hindu god does
quoting. Filenames are the lowercase deity, and **never carry a version or a date** —
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

Each app carries its own counter — they are not kept in step. Aengus starts at 26.0;
Kairos is already at 26.5, Minerva at 26.1 and Kubera at 26.2.

In Aengus the version lives in one place, `APP_VERSION`, and both the `<title>` and the
header chip are built from it. The other three still have it written out by hand in a
couple of spots (title, header chip, and the "Exit to …" button in Kairos and Minerva) —
worth collapsing to one constant next time each is touched.

## Aengus

Aengus is the front door. It opens listeners on the other three collections and shows:

- **On the clock** — who is clocked in right now, on what job, since when, plus hours
  logged today. From `kairos_state`.
- **The apps** — three plates linking to each app, each carrying its own live vitals.
- **Today's plan** — the published Minerva week, filtered to today. From `minerva_state`.
- **Quote pipeline** — draft/sent/won/lost counts and value. From `mercury_state`.

**Aengus is read-only and must stay that way.** It never writes a document. If the
dashboard could write, a stray click here could clobber a payroll entry or a live quote
with no way to tell which app did it. Anything that changes data belongs in Kairos,
Minerva or Kubera.

Links between the apps are relative (`kairos.html`, not a full URL), so the whole set
works unchanged whatever the site ends up being called, and works from a local folder too.

Two things Aengus is careful about, because getting them wrong is worse than useless on a
wall monitor: a panel that **could not be read** says so, rather than showing a
comforting zero; and a Minerva week that was published a while ago is labelled with its
age, so nobody plans a Tuesday off last month's board.

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
  the file would not move the data — it would just point the app at four empty documents
  and orphan every quote, pack list and archive row already stored.

New archive rows are stamped `source: 'Kubera'`. Rows already stamped `'Mercury'` are
displayed as Kubera so the column reads consistently. The stored value is left alone.

If the collection is ever renamed properly, it is a copy-then-cut-over, not a
find-and-replace: copy each `mc_*` doc to the new collection, verify, point the app at
it, keep the old collection read-only for a while, then delete.

## Moving to a lankosystems URL

Currently `https://ilaing-netizen.github.io/kairos-timeclock`. Two ways to get the name:

**A — new organisation (suggested).** Create a free GitHub organisation called
`lankosystems`, then a public repo inside it named exactly `lankosystems.github.io`. Your
personal account stays untouched, and you can add people later. Check
`github.com/lankosystems` first to confirm the name is free.

**B — rename the existing account.** Settings → Account → Change username, from
`ilaing-netizen` to `lankosystems`. Keeps all repos and history, but frees the old
username for anyone else to claim, and old links only redirect for a while.

Either way, the repo name `lankosystems.github.io` is what puts the site at the root:

```
https://lankosystems.github.io/            → index.html   (Aengus)
https://lankosystems.github.io/kairos.html
https://lankosystems.github.io/minerva.html
https://lankosystems.github.io/kubera.html
```

A repo named anything else, e.g. `apps`, lands the site at
`https://lankosystems.github.io/apps/` instead — which works, but the shorter root is
nicer to type on a tablet.

Then: push these four files to the default branch, Settings → Pages → Source: *Deploy
from a branch*, branch `main`, folder `/ (root)`. First deploy takes a minute or two.
Add the new domain to Firebase Console → Authentication → Settings → **Authorized
domains**, or anonymous sign-in will fail on the new URL and every app will show "no
connection". Re-bookmark the shop tablets — the old URL will not follow.

Note GitHub Pages needs a **public** repo unless you are on a paid plan. Public means
anyone with the URL loads the app, so:

## Two things worth tightening

Neither is caused by the rename, but both get more exposed on a new public URL.

**1. Anonymous auth is doing all the work.** Every app signs in anonymously and the rules
almost certainly say `request.auth != null`. That is satisfied by *anybody* who loads the
page, so in practice the shop's time entries, roster and quote values are readable and
writable by anyone with the URL. The app PINs are UI gates, not security — they are
checked in JavaScript the visitor controls. Worth moving to real accounts, or at minimum
rules that restrict writes per collection and make `mercury_state` unreadable except to
known users. Aengus only needs **read** on the three collections.

**2. Do not commit `Lanko_quote_archive.json`.** It is 640 KB of real customer quotes,
names and prices. In a public repo it is a public document. It belongs in Kubera's
importer or a private backup, not the site.
