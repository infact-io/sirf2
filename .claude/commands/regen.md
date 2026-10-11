# Regen

Regenerate all SIRF artefacts from the current schema.

1. Read `.claude/conventions.md` and `schemas/sirf.schema.json`.
2. Regenerate every file in the **Files** table in `.claude/CLAUDE.md` except `schemas/sirf.schema.json`, `CNAME` and `LICENSE`, so that each matches its description there and conforms to the current schema.
3. Follow the **Writing documentation** rules in `.claude/CLAUDE.md`, and never add a rule that isn't in `conventions.md`.
4. Run every step in **Checking a change** in `.claude/CLAUDE.md`, and report the results.
