# Weekly Brief — Update Guide

The live report is a single file, **`index.html`**, served at the root by Vercel. Each week you
update the data inside it, archive a snapshot, and push. Sensitive data folders stay local
(they're in `.gitignore` and never deploy).

---

## 1. Drop in the new week's data

Create a folder named for the week's date range and put this week's source files in it
(matching the existing pattern), e.g.:

```
6.08 to 6.12/
  EOW Notes.txt
  Files and Data Reference/
    Staff.Survey Responses.csv
    Careington EOB Tracking data.png
    Patient Survey Responses.*.png
    Budget Renovation Phase 2.png
```

These folders are git-ignored on purpose — they hold practice data and never get published.

## 2. Archive last week, then edit

```
# snapshot the current (last week's) report before you change it
mkdir "6.05 Report"            # name it for the Friday of the week it covers
cp index.html "6.05 Report/index.html"
```

Then edit `index.html`. Every editable region is wrapped in HTML comments so it's easy to find:

```html
<!-- DATA: 03 Patient Feedback -->
   ...the numbers and copy for that section...
<!-- /DATA -->
```

**Top-of-file bump points** (search for these):

| What | Where |
|------|-------|
| `Volume 24` → next number | masthead eyebrow |
| `Friday, June 5, 2026` → new date | masthead `m-date` |
| `WK 24` → next week number | appears in the table sub-header, `data-wk` on the Forward block, the signoff, and the Budget snapshot meta |
| `$56K` badge | Budget Snapshot tab button (round to the new Phase 2 total) |
| `const REPORT_ID = "wk24"` | comment script — **bump every week** so each report keeps its own comment thread |

**Section-by-section** — fill each `<!-- DATA -->` block from the data folder:

- **01 Insurance** — refresh the lead + bullet list; update the Careington table rows if line
  counts changed (from the EOB dashboard weekly summary).
- **02 Operations** — Overjet / verification status.
- **03 Renovation** — Phase 2 status bullets + the photo gallery (see "Photos & the assets
  folder" below).
- **04 Patient Feedback** — update the KPI `data-target`s (Responses, Avg = `data-target="488"
  data-decimal="100" data-places="2"` for 4.88), the four breakdown bars' `data-target` **and**
  `data-fill`, and the lead/micro-note (from the Patient Survey analysis).
- **05 Team** — rewrite the list + "My Read" from the Staff Pulse summary.
- **06 Marketing** — leads + campaign status + STS/DNS status (from `EOW Notes.txt`).
- **07 AI & Tools** — internal-tool updates and the EOB dashboard phase roadmap.
- **08 Forward** / **09 Asks** — next-week items and open decisions.
- **Budget tab** — banner total, cap, buffer, the six scope cards, and the context callout
  (from the Phase 2 budget xlsx, Executive Summary sheet).

### Photos & the assets folder

Deployable images live in the tracked **`assets/`** folder (the weekly data folders are
git-ignored and never deploy, so anything the page references must be copied there).
Process new photos with `sips` before copying — keep the page light:

```
# concepts/photos → JPEG, max 1600px
sips -s format jpeg -s formatOptions 80 -Z 1600 "Source.png" --out assets/slug-name.jpg
# line art (floor plans) → keep PNG
sips -Z 1600 "Floor Plan.png" --out assets/floor-plan.png
```

Reference them with relative paths (`assets/foo.jpg`), give every `<img>` a real `alt` and
`loading="lazy"`, and wrap it in `<a href="assets/foo.jpg" target="_blank" rel="noopener">`
so clicking opens the full image. The gallery markup is the `.photo-grid` block in
**03 Renovation** (`.photo-card.wide` = full-row, uncropped — used for the floor plan).

## 3. Publish

```
git add index.html UPDATE_GUIDE.md .gitignore
git commit -m "Weekly Brief WK 25 (6.08–6.12)"
git push
```

Vercel auto-deploys the root `index.html`. (The archive + data folders are git-ignored, so
they won't deploy — that's intended.)

---

## Comments setup (one-time)

The report has a shared comment thread under every section so Dr. Friend can leave notes you'll
see on any device. Comments live in **Supabase** (free). Until it's configured the comment UI
simply stays hidden and the report works normally.

**1. Create the project.** Sign up at [supabase.com](https://supabase.com) → New project (free
tier). Note the **Project URL** and **anon public key** under *Project Settings → API*.

**2. Create the table.** In the Supabase **SQL Editor**, run:

```sql
create table public.comments (
  id          bigint generated always as identity primary key,
  report_id   text not null,
  section_id  text not null,
  author      text not null,
  body        text not null,
  created_at  timestamptz not null default now()
);
alter table public.comments enable row level security;
create policy "read"   on public.comments for select using (true);
create policy "insert" on public.comments for insert with check (true);
-- live updates:
alter publication supabase_realtime add table public.comments;
```

**3. Wire it up.** In `index.html`, find the comment config near the bottom and paste your
values:

```js
const REPORT_ID = "wk24";                       // bump each week
const SUPABASE_URL = "https://xxxx.supabase.co";
const SUPABASE_ANON_KEY = "ey...";              // anon / public key
```

Commit + push. Done — comments now persist and sync across devices.

**Good to know:**
- Comments are scoped by `REPORT_ID`, so bumping it each week gives every report its own clean
  thread. Past weeks' comments stay attached to past weeks (visible if you reopen that archived
  file with the same REPORT_ID).
- The anon key is *meant* to be public (row-level security limits it to the `comments` table) —
  it's safe to commit. But because the report URL is public, anyone with the link could read or
  add comments. Fine for an internal brief; if you ever want it locked down, we can add a shared
  passphrase gate.
- Each person picks their name once ("Joshua Moon" / "Dr. Friend" / custom); it's remembered in
  their browser.
