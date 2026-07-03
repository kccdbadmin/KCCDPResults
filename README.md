# IB DP Results Analyzer

A single-page web app for IB Diploma Programme results day. Drop the official IBIS
**"Results summary" PDF** onto the page and get an analysis dashboard for school
leadership, a multi-sheet Excel export, and a print-friendly leadership summary.

**Privacy by design:** the PDF is parsed entirely in the browser with
[pdf.js](https://mozilla.github.io/pdf.js/). Nothing is uploaded, fetched, or stored —
student data never leaves the page. This makes it safe to host on GitHub Pages or to
open `index.html` straight from disk.

## Use

1. Open `index.html` (locally or via GitHub Pages).
2. Drop the IBIS Results summary PDF (one candidate per page, Oracle Reports format).
3. Review the dashboard:
   - KPI cards — pass rate, average points, highest score, 40+ scorers, bilingual diplomas, average subject grade
   - Highlights — top performer(s), 40+/30+ counts (IB convention: ≥40 / ≥30), grade-7 tally
     and straight-7 students, strongest subjects and subjects to review (≥5 entries),
     group-gap callouts (with masterlist), and a **remark watchlist** of candidates who
     missed the diploma by ≤2 points or failed a condition at 24+ — the enquiry-upon-results
     shortlist. Names follow the Anonymize toggle everywhere, including print.
     A world-average selector compares the school against the official IB mean total
     points: figures for May 2021–2025 are embedded from the IB's *DP and CP Final
     Statistical Bulletin – May 2025* (ibo.org, p. 8, published to 1 dp), auto-selected
     to match the parsed session, drawn as a second marker on the points-distribution
     chart with a source footnote below it (dashboard, print, Excel). A Custom… option
     accepts a manual value (labelled as unofficial). Runtime auto-fetch from ibo.org is
     not possible from a static page (no API/CORS; the site also blocks bots), so new
     sessions are added as one line in `WORLD_AVERAGES` in `index.html` when the next
     bulletin is published.
   - Total points distribution with mean marker
   - Subject performance table (sortable, HL/SL filter, grade mini-bars)
   - Grade distributions (1–7 by level, plus EE and TOK letter grades)
   - "Diploma not awarded" panel with the parsed diploma-requirements reason
   - Full searchable candidate table
4. **Anonymize names** toggle for projecting in meetings (names become candidate numbers).
5. **Export Excel** — one workbook with `Candidates`, `Subjects` (pivot-ready long format),
   `Subject Summary`, `Points Distribution`, and `Summary` sheets.
6. **Print leadership summary** — a print stylesheet turns the dashboard into a ~2-page
   A4 handout (student names are never printed).

## Student masterlist (gender / nationality breakdowns)

Drop an MIS export (e.g. iSAMS, `.xlsx`/`.csv`) alongside the PDF — together in the
dropzone or later via **Load masterlist…** — to attach demographics to candidates and
unlock the **Group comparisons** panel (Male vs Female, Emirati vs Non-Emirati,
nationality) plus `Group Comparison` and `Match Review` sheets in the Excel export.
The masterlist is parsed client-side too; nothing is uploaded or stored.

Columns are auto-detected by header name (`Full Name`/`Forename`/`Surname`,
`Date of Birth`, `Gender`, `Nationality`, and optionally an IB
`Candidate No`/`Personal Code` column). Rows are joined to candidates by, in order:

1. **Candidate/personal code** — exact (add this column to future exports for zero-risk joins);
2. **DOB + surname + first given name** — names normalized for case, accents, spacing
   and hyphens (`Al Hashmi` = `AlHashmi`);
3. **DOB + surname** or **DOB + first name** alone, and **name-only** (when the
   masterlist has no DOB) — flagged for review, never trusted silently.

DOB alone is never sufficient, and a same-name row with a different DOB is reported,
not matched. Every non-exact join and every unmatched candidate is listed in the
**Masterlist match** review panel; unmatched candidates appear as "Unknown" in group
charts so nothing silently disappears. If fewer than half the candidates match, a
prominent warning flags that the masterlist probably doesn't cover the cohort
(wrong campus / year / stale export).

"Emirati" is derived from the Nationality column: a student counts as Emirati if
*any* listed nationality (comma-separated) is `Emirati` / `United Arab Emirates` /
`UAE`; blank nationality becomes "Unknown". The nationality view groups by the
first-listed (primary) nationality, top 7 + Other.

Masterlists are **never committed** (`*.xlsx`/`*.csv` are gitignored) and never
persisted by the app — load the file each session.

## Format notes

The parser reconstructs text lines from pdf.js coordinates (grouping by `y`, ordering
by `x`), so it does not depend on extraction order or whitespace. It is format-driven,
not sample-specific: session (May/November), school code, categories
(`DIPLOMA` / `COURSE` / `RETAKE` / others), grades `1–7`, `A–E`, `N`, retake
"Additional subjects" blocks, missing CAS lines, and multi-line failure reasons are all
handled. Anything unrecognized is reported in a visible **parse warnings** panel instead
of failing silently.

Truncated Oracle subject names (e.g. `MATHEMATICS ANALYSIS AND APPRO`) are prettified
via the `SUBJECT_NAMES` mapping table at the top of the script in `index.html` — edit it
there to adjust display names. Raw names are always kept alongside.

## Testing against the fixture

Real results PDFs are **deliberately not committed** (see `.gitignore`). To run the
parser validation suite, place the fixture `May_2025_results_RAHA.pdf` next to
`index.html`, serve the folder locally, and open with `?test=1`:

```
python -m http.server 8000
# then browse to http://localhost:8000/index.html?test=1
```

The page parses the fixture and asserts the known counts (141 candidates,
136/4/1 by category, 115/5/17 by result, 137 CAS satisfied, max 43 points, …),
showing PASS/FAIL per check.

## Stack

Plain HTML/CSS/JS in one file. CDN dependencies: pdf.js 3.11 (text extraction),
Chart.js 4 (charts), SheetJS (xlsx export). No build step, no server.
