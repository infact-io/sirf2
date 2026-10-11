# SIRF — Standard Industry Reporting Format v2

This repo defines SIRF, a JSON Lines format for consumer credit data reporting: a JSON Schema (draft 2020-12), sample data, and documentation. It is published at https://sirf2.org by GitHub Pages from `main` (root folder). Every push to `main` updates the site.

The site uses the Just the Docs theme. Web addresses: `README.md` → `/`, `sirf-schema.md` → `/format/`, `sirf-delivery.md` → `/delivery/`, `sirf-validation.md` → `/validation/` (the old `.html` addresses redirect via the pages in `redirects/`). Page titles, menu order and addresses are set in `_config.yml`, not in front matter, so don't add front matter to the Markdown files. A new documentation page needs its own entry under `defaults` in `_config.yml`. Links between pages stay as relative `.md` links; GitHub Pages rewrites them to the new addresses.

## Rules

- **`.claude/conventions.md` is the only source of business rules.** Read it before any schema, sample or documentation change.
- **Never infer rules.** Do not add a constraint from domain knowledge, industry practice or common sense, even when it seems obvious (submission frequency, age limits, retention, lifecycle, field reuse and so on). If a rule isn't in `conventions.md` and the user hasn't confirmed it:
  - keep it out of schema `description` fields and the documentation;
  - raise it as an open question for the user to decide.
- **Record every decision the user makes in `conventions.md`**, in the same change as the schema or docs, so the rules never live only in the schema.
- When a new field looks categorical, propose enum values and ask the user to confirm them.
- The user is a CTO with deep credit-data expertise: be direct and technical, focus on trade-offs and edge cases, and say plainly when something in the schema or data could be better.

## Files

Keep all of these consistent with the schema. `/regen` regenerates the generated ones.

| File | Contents |
| --- | --- |
| `schemas/sirf.schema.json` | The schema, applied per line. `$defs/header` (line 1), `$defs/accountRecord` (lines 2+), `$defs/account` (nested account object with the conditional rules), `$defs/flag`, `$defs/person`, `$defs/address`. `$id` is `https://sirf2.org/schemas/sirf.schema.json`. |
| `README.md` | Minimal narrative and links to the files below in three tables, in this order: Documentation, Schemas, Samples (relative links). No licence mention. Ends with a line saying SIRF is an open source format, linking to https://github.com/infact-io/sirf2. |
| `sirf-schema.md` | Format reference: file structure, Header, Account Record, Account, Person, Address, enumeration values, conventions, examples. Examples section: pretty-printed Header first, then pretty-printed individual account examples, then the batch file as JSON Lines. No changelog section. |
| `sirf-delivery.md` | Delivery: JSON Lines files for bulk reporting, and single Account records sent by HTTP `POST /report` for real-time reporting and individual corrections. Its example file is identical to `samples/sample-batch.jsonl`. POST examples are a `text` block (request line and header) followed by a `json` block (body). |
| `sirf-validation.md` | Validating with JSON Schema: what the schema checks and can't check, error examples, and one Python validator (`jsonschema`) with the file-level checks. Error messages and example output are real `jsonschema` output. |
| `samples/sample-instance-mortgage.json` | Single Mortgage account record (with `accountSubtype`), pretty-printed. Carries `account.portfolioId`. |
| `samples/sample-instance-unsecured-loan.json` | Single UnsecuredLoan account record, pretty-printed. Carries `account.portfolioId`. |
| `samples/sample-instance-credit-card.json` | Single CreditCard account record (with `creditLimit`, `minimumPayment`, `cashAdvances`, `cashAdvancesCount`), pretty-printed. Carries `account.portfolioId`. |
| `samples/sample-batch.jsonl` | 1 Header + 5 Account records, one compact JSON object per line. No `account.portfolioId`. |
| `_config.yml` | GitHub Pages site settings: Just the Docs theme (`remote_theme`), search, GitHub link, footer, and each page's title, menu order and web address (in `defaults`). Not regenerated. |
| `redirects/*.html` | Redirect pages from the old addresses (`/sirf-schema.html` and so on) to the new ones. Not regenerated. |
| `CNAME` | `sirf2.org`, used by GitHub Pages. Don't remove it. |
| `LICENSE` | MIT licence. Not regenerated. |

Samples use realistic but clearly fictional UK data and cover a variety of account types, statuses, address shapes and flags (see "Sample Data Conventions" in `conventions.md`).

## Writing documentation

- Write for a semi-technical audience: lenders' and receivers' data and engineering teams.
- Individual records are pretty-printed with a 2-space indent, one example per code block.
- **Never wrap or split lines in JSON Lines examples.** Each record is one unbroken line, however long.
- Only assert what `conventions.md` or the user has stated.

## Checking a change

After changing the schema, samples or docs:

1. Validate every sample, every schema `examples` entry, and every record in the docs' `json` blocks against the schema (`$defs/header` for headers, `$defs/accountRecord` for accounts).
2. Run the Python validator from `sirf-validation.md` against `samples/sample-batch.jsonl` (expect `valid`), and against a copy with deliberate mistakes to confirm the example output on the page still matches exactly.
3. Confirm the batch file in `sirf-delivery.md` and `sirf-schema.md` is identical to `samples/sample-batch.jsonl`.

## Git

Commit and push only when the user asks. After pushing, the Pages build takes a minute or two; check it with `gh api repos/infact-io/sirf2/pages/builds/latest`.
