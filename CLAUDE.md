# Project conventions

## Secrets policy (standing rule — always enforce)

Every time a new tool, API, database, or service is connected, or a key/token/
password/connection string appears in any form (pasted by the user, generated
during setup, found in code):

1. Put the real value in `.env` as `NAME=value` on its own line, with a clear
   descriptive name (e.g. `KOBO_API_TOKEN`, `DATABASE_URL`).
2. Reference it from code via the environment variable — never hardcode it.
3. Add the same variable name with a blank/placeholder value to `.env.example`.
4. Never commit `.env` (it is gitignored; keep it that way). Verify with
   `git check-ignore .env` before any commit that touches env files.
5. If a secret is ever found already committed, tell the user exactly which
   secret and file, and remind them to rotate it at the source — gitignoring
   after the fact does not un-leak it.

Additional constraints specific to this project:

- `index.html` is a fully offline, client-side app opened directly in a
  browser. It cannot read environment variables, and anything inside it is
  visible to everyone who has the file. Never place a secret in `index.html`;
  if a feature ever needs a secret, it needs a server or build step first.
- Beneficiary data is sensitive PII. The app's CSV exports
  (`beneficiaries-export-*.csv`, `duplicate-report-*.csv`) are gitignored;
  never commit beneficiary data files of any kind.

## Verification

End-to-end tests (Playwright + system Chromium at /opt/pw-browsers) live in
the session scratchpad, not the repo. Core checks when changing index.html:
registration + live duplicate check, CSV import round-trip, duplicate scan
against demo data with planted duplicates, and a large-scale scan benchmark.
