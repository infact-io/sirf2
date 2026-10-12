# Standard Industry Reporting Format (SIRF)

A JSON Lines format for consumer credit data reporting.

> This is an early draft proposal of the new format and its definition, focusing on the approach rather than the detail. There is a significant amount of development and change to come.
{: .early }

## Documentation

| File | Description |
| --- | --- |
| [sirf-schema.md](sirf-schema.md) | Format reference — field tables, enum values, conventions |
| [sirf-delivery.md](sirf-delivery.md) | Delivering SIRF data — JSON Lines files for bulk reporting, single records over HTTP POST for real-time reporting and corrections |
| [sirf-validation.md](sirf-validation.md) | Validating SIRF files with JSON Schema — what is checked, error examples, Python validator |

## Schemas

| File | Description |
| --- | --- |
| [schemas/sirf.schema.json](schemas/sirf.schema.json) | JSON Schema (draft 2020-12) for all SIRF records (header and account) |

## Samples

| File | Description |
| --- | --- |
| [samples/sample-batch.jsonl](samples/sample-batch.jsonl) | Complete SIRF file — 1 header + 5 account records |
| [samples/sample-instance-mortgage.json](samples/sample-instance-mortgage.json) | Single Mortgage account record |
| [samples/sample-instance-unsecured-loan.json](samples/sample-instance-unsecured-loan.json) | Single UnsecuredLoan account record |
| [samples/sample-instance-credit-card.json](samples/sample-instance-credit-card.json) | Single CreditCard account record |

---

SIRF is an open source format. The source is on GitHub: [github.com/infact-io/sirf2](https://github.com/infact-io/sirf2).
