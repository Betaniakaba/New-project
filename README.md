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
  - identical **National ID** → flagged as a sure duplicate (values shared by
    very many records — placeholders like "NA" or "00000" — are recognized
    and never mass-merge strangers);
  - **nearly identical National IDs** (a one-digit typo) count as strong
    evidence instead of hiding an otherwise perfect match;
  - fuzzy **name matching** (phonetic keys with Amharic-romanization folding
    such as Tsegaye/Segaye or Qedir/Kedir, plus token-level Jaro-Winkler)
    that catches typos, alternative spellings and **swapped name order** —
    while one *genuinely different* name part (siblings, twins) lowers the
    score;
  - tolerant **date-of-birth comparison**: day/month swaps and off-by-one
    years count as near-matches, and default "January 1" birthdays are
    treated as weak evidence;
  - corroborated by **phone, sex, location and household ID**;
  - contradiction penalties (clearly different national IDs, birth years far
    apart) and evidence scaling (missing data lowers confidence) to keep
    false positives down;
  - blocking (phonetic name key and name-part pairs / DOB / phone / ID /
    household) so the scan compares only plausible candidate pairs, with
    oversized blocks split by birth year and sex rather than dropped —
    ~100,000 records scan in seconds, and million-record datasets remain
    tractable;
  - "probable" matches are grouped; "possible" matches are reported as
    stand-alone pairs and deliberately never chained, so unrelated people
    can't merge into one giant group.
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
- **Block cap** — comparison blocks larger than this are split by birth year
  and sex first; only what still doesn't fit is skipped (exact national-ID
  matches are never skipped).

## Known trade-offs

- **Twins vs. re-registrations**: two records that agree on everything except
  one name part (same DOB, same household) look identical whether they are
  twins or the same person recorded under a different naming convention. The
  engine surfaces them as "possible" pairs for human review rather than
  merging them — unless the National IDs agree, which settles it.
- **The live registration check is a quick screen** (exact ID/phone/DOB and
  phonetic-name candidates). The full scan in the Duplicates tab uses wider
  blocking and is the authoritative cross-reference — run it after imports
  and periodically.
- National-ID values shared by more than 25 records are treated as
  placeholders (e.g. "NA", "0000000") and ignored as identifiers; the scan
  summary reports how many such values were found.
- Ethiopian-calendar dates are not converted; datasets mixing EC and
  Gregorian birth dates should be converted to one calendar before import.

## Important notes

- Data is per-browser, per-computer. To move or back up data, use
  **Export CSV** — and export regularly: clearing the browser's site data
  would erase the registry.
- Dates are normalized to `YYYY-MM-DD` on import; `DD/MM/YYYY` is assumed for
  ambiguous day/month values.
- Phone numbers are compared on their last 9 digits, so `+2519…`, `09…` and
  `9…` formats match each other.
