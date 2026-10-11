# Standard Industry Reporting Format (SIRF)

A JSON Lines format for consumer credit data reporting.

MIT licence — see [LICENSE](LICENSE) for details.

## Schemas

| File | Description |
| --- | --- |
| [schemas/sirf.schema.json](schemas/sirf.schema.json) | JSON Schema (draft 2020-12) for all SIRF records (header and account) |

## Documentation

| File | Description |
| --- | --- |
| [sirf-schema.md](sirf-schema.md) | Format reference — field tables, enum values, conventions |

## Samples

| File | Description |
| --- | --- |
| [samples/sample-batch.jsonl](samples/sample-batch.jsonl) | Complete SIRF file — 1 header + 5 account records |
| [samples/sample-instance-mortgage.json](samples/sample-instance-mortgage.json) | Single Mortgage account record |
| [samples/sample-instance-unsecured-loan.json](samples/sample-instance-unsecured-loan.json) | Single UnsecuredLoan account record |
| [samples/sample-instance-credit-card.json](samples/sample-instance-credit-card.json) | Single CreditCard account record |
