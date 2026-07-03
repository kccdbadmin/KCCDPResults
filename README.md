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
