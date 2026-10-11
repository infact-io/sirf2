# Regen

Regenerate all SIRF artefacts from the current schema files.

Read the following files first to get the current state:

- `.claude/skills/credit-schema-designer/conventions.md`
- `schemas/sirf.schema.json`

Then regenerate all of the following:

1. `README.md` — top-level repo readme: minimal narrative, MIT licence notice, relative links to all generated files in three tables (Documentation, Schemas, Samples)
2. `sirf-schema.md` — full format reference documentation (pretty-printed Header example first, then pretty-printed individual account examples; batch example as JSON Lines)
3. `sirf-delivery.md` — delivery guide: JSON Lines files and single-record HTTP POST. Keep its example file identical to `samples/sample-batch.jsonl` and its POST bodies valid against `$defs/accountRecord`
4. `sirf-validation.md` — guide to validating SIRF files with JSON Schema. Update its rule tables, error examples and validator code so they match the current schema, and re-run the Python validator to confirm the example output is still accurate
5. `samples/sample-instance-mortgage.json` — pretty-printed Mortgage account record (with `accountSubtype`) conforming to the current schema
6. `samples/sample-instance-unsecured-loan.json` — pretty-printed UnsecuredLoan account record conforming to the current schema
7. `samples/sample-instance-credit-card.json` — pretty-printed CreditCard account record (with `creditLimit`, `cashAdvances`, `cashAdvancesCount`) conforming to the current schema
8. `samples/sample-batch.jsonl` — 6 records (1 header + 5 accounts) conforming to the current schema, one JSON object per line

Follow all rules in `.claude/skills/credit-schema-designer/conventions.md` including the no-inference rule.
