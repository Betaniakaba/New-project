# Beneficiary Registry & Duplicate Finder

A single-file, fully offline web application for registering beneficiaries,
storing large datasets **on your own computer**, and cross-referencing the
whole dataset to find duplicate registrations.

## How to use it

1. Download `index.html` (or clone this repository).
2. Double-click `index.html` — it opens in your browser (Chrome/Edge/Firefox).
3. That's it. No server, no installation, no internet connection needed.

All data is stored in the browser's built-in local database (IndexedDB) on the
computer where you open the page. Nothing is ever sent anywhere.

## What it does

- **Register** beneficiaries with the standard fields: names (first / father's /
  grandfather's), sex, date of birth, National ID, phone, household ID and size,
  Region / Zone / Woreda / Kebele, program, registration date, notes.
- **Live duplicate check** — every new registration is checked against all
  existing records *before* it is saved, and you are shown the likely matches.
- **Import CSV** — bring in your existing datasets, whatever the column layout:
  you map your CSV columns to the registry fields (with auto-guessing from your
  headers). Files are read as a stream, so very large files (hundreds of
  thousands to millions of rows) import without freezing the browser.
- **Full-dataset duplicate scan** — cross-references every record against the
  rest of the dataset in a background worker:
  - identical **National ID** → flagged as a sure duplicate;
  - fuzzy **name matching** (phonetic keys + token-level Jaro-Winkler) that
    catches typos, alternative spellings and **swapped name order**;
  - corroborated by **date of birth, phone, sex and location**;
  - contradiction penalties (different national IDs, different birth years)
    and evidence scaling (missing data lowers confidence) to keep false
    positives down;
  - blocking (phonetic name key / DOB / phone / ID) so the scan compares only
    plausible candidate pairs — ~100,000 records scan in seconds, and
    million-record datasets remain tractable.
- **Review workflow** — duplicate groups ranked by confidence ("probable" vs
  "possible"), side-by-side record comparison, delete records, mark groups as
  reviewed, and export a full duplicate report to CSV.
- **Search & browse** — paginated table with search by name, National ID or phone.
- **Export** — full registry or duplicate report as CSV at any time.
- **Demo data generator** (Settings tab) — creates sample records with planted
  duplicates so you can try the scan before importing real data.

## Matching sensitivity

Tune in **Settings & Tools**:

- **Probable threshold** (default 0.82) — pairs at or above are treated as
  strong matches and merged into groups.
- **Possible threshold** (default 0.70) — pairs between the two thresholds are
  reported as stand-alone pairs for review (they are deliberately never
  chained together, to avoid unrelated people being merged into one group).
- **Block cap** — very common name blocks larger than this are skipped for
  speed (exact ID matches are never skipped).

## Important notes

- Data is per-browser, per-computer. To move or back up data, use
  **Export CSV** — and export regularly: clearing the browser's site data
  would erase the registry.
- Dates are normalized to `YYYY-MM-DD` on import; `DD/MM/YYYY` is assumed for
  ambiguous day/month values.
- Phone numbers are compared on their last 9 digits, so `+2519…`, `09…` and
  `9…` formats match each other.
