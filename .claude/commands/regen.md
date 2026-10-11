# Regen

Regenerate all SIRF artefacts from the current schema files.

Read the following files first to get the current state:

- `.claude/skills/credit-schema-designer/conventions.md`
- `schemas/sirf.schema.json`

Then regenerate all of the following:

1. `README.md` — top-level repo readme: minimal narrative, MIT licence notice, relative links to all generated files in three tables (Schemas, Documentation, Samples)
2. `sirf-schema.md` — full format reference documentation
3. `samples/sample-instance-mortgage.json` — pretty-printed Mortgage account record (with `accountSubtype`) conforming to the current schema
4. `samples/sample-instance-unsecured-loan.json` — pretty-printed UnsecuredLoan account record conforming to the current schema
5. `samples/sample-instance-credit-card.json` — pretty-printed CreditCard account record (with `creditLimit`, `cashAdvances`, `cashAdvancesCount`) conforming to the current schema
6. `samples/sample-batch.jsonl` — 6 records (1 header + 5 accounts) conforming to the current schema, one JSON object per line

Follow all rules in `.claude/skills/credit-schema-designer/conventions.md` including the no-inference rule.
