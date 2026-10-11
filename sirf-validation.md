# Standard Industry Reporting Format (SIRF) — Validating with JSON Schema

SIRF publishes its rules as a [JSON Schema](https://json-schema.org/) so that lenders can check their files before submitting them, and receivers can check files as they arrive, both against exactly the same rules. This page explains how the SIRF schema works, what it does and doesn't check, and how to validate a SIRF file in Python.

For the fields themselves, see the [Format Reference](sirf-schema.md).

---

## Why JSON Schema

JSON Schema is an open standard for describing the shape of JSON data: which fields exist, what type each one is, which are required, and which values are allowed. Describing SIRF this way has some practical benefits:

- **One set of rules for everyone.** Lenders and receivers validate against the same schema file, so a file that passes before it's sent also passes when it arrives.
- **Off-the-shelf tools.** Mature validators exist for every mainstream language, so nobody needs to write their own SIRF validation logic.
- **Errors that point to the problem.** A validator reports the field that failed and the rule it broke, such as `address.postalCode: must match pattern …`, rather than simply rejecting the file.
- **The rules are documentation.** Every field in the schema has a title, a description and example values, so the schema can be read as well as run.

---

## The Schema

| | |
| --- | --- |
| **File** | [schemas/sirf.schema.json](schemas/sirf.schema.json) |
| **Published at** | `https://sirf2.org/schemas/sirf.schema.json` (also the schema's `$id`) |
| **JSON Schema version** | Draft 2020-12 |

A SIRF file is a [JSON Lines](https://jsonlines.org/) file: one JSON object per line. The schema describes **a single line**, so a file is validated one line at a time.

The schema is organised into reusable definitions under `$defs`:

| Definition | Validates |
| --- | --- |
| `$defs/header` | The Header record on line 1 |
| `$defs/accountRecord` | Each Account record on lines 2 onwards |
| `$defs/person` | The `person` object inside an account record |
| `$defs/address` | The `address` object inside an account record |
| `$defs/account` | The `account` object inside an account record, which holds all the account data |

The schema's root accepts either a header or an account (`oneOf`). That's convenient for a quick check of a single line, but when a line fails, the validator can't tell which of the two record types you meant, and the error messages are vague. **Validate each line against `$defs/header` or `$defs/accountRecord` directly**, depending on its position in the file. The examples below do this.

---

## What the Schema Checks

| Rule | How it's expressed | SIRF example |
| --- | --- | --- |
| Required fields | `required` | Every account must have `accountId`, `accountType` and `status`. |
| Field types | `type` | Monetary amounts must be whole numbers (`integer`), not decimals. |
| Allowed values | `enum`, `const` | `status` must be one of the listed statuses; `sirfVersion` must be `"2.0"`. |
| Text formats | `pattern` | Postcodes need their internal space (`LS6 2AB`); `accountId` is letters, digits and hyphens. |
| Dates | `format: "date"` plus `pattern` | Dates must be zero-padded `YYYY-MM-DD`. |
| Minimum values | `minimum` | Balances and repayments can't be negative; `recordCount` is at least 1. |
| No unknown fields | `additionalProperties: false` | A misspelt or invented field such as `balance` is rejected, as is an account field placed outside the `account` object. |
| Rules that depend on other fields | `if` / `then` / `else` | Card accounts need `creditLimit`; a delinquent account needs `daysPastDue`. |

### The conditional rules

Some fields in the `account` object are required or prohibited depending on the values of other fields. The schema expresses these with `if` / `then` / `else` in `$defs/account`:

| When | Then |
| --- | --- |
| `accountType` is `CreditCard`, `ChargeCard` or `Budget` | `creditLimit` is required |
| `accountType` is `CreditCard` | `minimumPayment` is also required |
| `accountType` is anything else | `creditLimit`, `minimumPayment`, `cashAdvances` and `cashAdvancesCount` must not be present |
| `status` is `Delinquent1` to `Delinquent6` | `daysPastDue` is required |

For example, this is how the schema says that delinquent accounts must report `daysPastDue`:

```json
{
  "if": {
    "properties": { "status": { "enum": ["Delinquent1", "Delinquent2", "Delinquent3", "Delinquent4", "Delinquent5", "Delinquent6"] } },
    "required": ["status"]
  },
  "then": { "required": ["daysPastDue"] }
}
```

### What the schema can't check

JSON Schema looks at one record at a time, so rules about **the file as a whole** need a few lines of your own code:

- Line 1 must be the Header record, and every other line must be an Account record.
- `recordCount` in the header must equal the number of account records in the file.
- Account records in a file must not include `account.portfolioId`. That field is only for single records sent by HTTP POST; in a file, the header's `portfolioId` applies. See [Delivery](sirf-delivery.md).

The example validator below includes these checks.

---

## Examples of Errors

Each example below starts from a valid account record and changes one thing. Only the relevant part of the record is shown. The error shown is the message from the Python [`jsonschema`](https://python-jsonschema.readthedocs.io/) library, used in the [example validator](#validating-a-file) further down. Other validators word their messages differently but report the same problems.

### Missing a required field

The account has no `status`:

```json
{ "account": { "accountId": "MBL-7722", "accountType": "CreditCard", "startDate": "2020-11-01" } }
```

```text
account: 'status' is a required property
```

### A decimal monetary amount

Monetary amounts are whole pounds, so `185.5` must be reported as `186`:

```json
{ "account": { "repayment": 185.5 } }
```

```text
account.repayment: 185.5 is not of type 'integer'
```

### A value that isn't in the allowed list

```json
{ "account": { "status": "Late" } }
```

```text
account.status: 'Late' is not one of ['Unclassified', 'Dormant', 'UpToDate', 'Delinquent1', 'Delinquent2', 'Delinquent3', 'Delinquent4', 'Delinquent5', 'Delinquent6', 'Defaulted']
```

### A postcode without its internal space

```json
{ "address": { "line1": "Birchwood Avenue", "postalCode": "LS62AB" } }
```

```text
address.postalCode: 'LS62AB' does not match '^[A-Z]{1,2}\\d[A-Z\\d]?\\s\\d[A-Z]{2}$'
```

### A date in the wrong format

Dates must be zero-padded `YYYY-MM-DD`, so `2020-11-1` fails and `2020-11-01` passes:

```json
{ "account": { "startDate": "2020-11-1" } }
```

```text
account.startDate: '2020-11-1' is not a 'date'
account.startDate: '2020-11-1' does not match '^\\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\\d|3[01])$'
```

### A field that SIRF doesn't define

```json
{ "account": { "balance": 2340 } }
```

```text
account: Additional properties are not allowed ('balance' was unexpected)
```

### An account field outside the `account` object

All account data belongs inside `account`. An account field at the top level of the record is rejected as unknown:

```json
{ "recordType": "Account", "person": { "...": "..." }, "address": { "...": "..." }, "accountId": "MBL-7722" }
```

```text
(record): Additional properties are not allowed ('accountId' was unexpected)
(record): 'account' is a required property
```

### A card field on a non-card account

A `Mortgage` account must not carry `creditLimit`:

```json
{ "account": { "accountType": "Mortgage", "creditLimit": 5000 } }
```

```text
account.creditLimit: False schema does not allow 5000
```

"False schema does not allow" is how `jsonschema` reports a field that the schema prohibits in this situation.

### A delinquent account without days past due

```json
{ "account": { "accountType": "UnsecuredLoan", "status": "Delinquent2" } }
```

```text
account: 'daysPastDue' is a required property
```

---

## Validating a File

The example below validates a whole SIRF file: it checks line 1 against `$defs/header`, every other line against `$defs/accountRecord`, `recordCount` against the number of account records, and that no account record carries its own `portfolioId`. Run it from the root of this repository so that `schemas/sirf.schema.json` can be found.

### Python

Uses the [`jsonschema`](https://python-jsonschema.readthedocs.io/) library. The `[format]` extra lets it check `format: "date"`.

```text
pip install "jsonschema[format]"
```

```python
import json
import sys

from jsonschema import Draft202012Validator

with open("schemas/sirf.schema.json", encoding="utf-8") as f:
    schema = json.load(f)

header = Draft202012Validator(
    {"$ref": "#/$defs/header", "$defs": schema["$defs"]},
    format_checker=Draft202012Validator.FORMAT_CHECKER,
)
account = Draft202012Validator(
    {"$ref": "#/$defs/accountRecord", "$defs": schema["$defs"]},
    format_checker=Draft202012Validator.FORMAT_CHECKER,
)

# Card-only fields, which the schema prohibits on other account types.
CARD_ONLY = [k for k, allowed in schema["$defs"]["account"]["else"]["properties"].items() if allowed is False]

def field_name(err, record):
    path = list(err.absolute_path)
    if err.validator is None:
        # jsonschema reports a prohibited field against its parent object,
        # so name the card-only field(s) that are present.
        parent = record
        for key in path:
            parent = parent[key]
        path.append(", ".join(k for k in CARD_ONLY if k in parent))
    return ".".join(str(p) for p in path) or "(record)"

def check_file(path):
    problems = []
    accounts = 0
    with open(path, encoding="utf-8") as f:
        for line_no, line in enumerate(f, start=1):
            if not line.strip():
                continue
            record = json.loads(line)
            # Line 1 must be the header; every other line must be an account.
            v = header if line_no == 1 else account
            for err in v.iter_errors(record):
                field = field_name(err, record)
                problems.append(f"line {line_no}: {field}: {err.message}")
            if line_no == 1:
                expected = record.get("recordCount")
            else:
                accounts += 1
                # In a file, the header's portfolioId applies to every account.
                if "portfolioId" in record.get("account", {}):
                    problems.append(f"line {line_no}: account.portfolioId: not used in JSON Lines files")
    if accounts != expected:
        problems.append(f"recordCount is {expected} but file has {accounts} account records")
    return problems

problems = check_file(sys.argv[1])
print("\n".join(problems) or "valid")
sys.exit(1 if problems else 0)
```

```text
python validate_sirf.py samples/sample-batch.jsonl
```

### Example output

For the [sample batch file](samples/sample-batch.jsonl), the script prints `valid` and exits with code `0`.

Here is the output for a copy of that file with six mistakes added: a `creditLimit` on a mortgage, a postcode without its space, a decimal repayment, a delinquent loan with no `daysPastDue`, an unknown status, and the last account removed so the header's `recordCount` is wrong:

```text
line 2: account.creditLimit: False schema does not allow 5000
line 3: address.postalCode: 'LS62AB' does not match '^[A-Z]{1,2}\\d[A-Z\\d]?\\s\\d[A-Z]{2}$'
line 3: account.repayment: 185.5 is not of type 'integer'
line 4: account: 'daysPastDue' is a required property
line 5: account.status: 'Late' is not one of ['Unclassified', 'Dormant', 'UpToDate', 'Delinquent1', 'Delinquent2', 'Delinquent3', 'Delinquent4', 'Delinquent5', 'Delinquent6', 'Defaulted']
recordCount is 5 but file has 4 account records
```

The exit code is `1`, so the script can be used to stop a submission pipeline when a file is invalid.

---

## Tips

- **Check dates with `format` enabled.** `jsonschema` doesn't check `format: "date"` unless you pass a `format_checker`, as the example does. SIRF also checks dates with a `pattern`, so the layout is always enforced, but only format checking rejects impossible dates such as `2025-02-30`.
- **Use a validator that supports Draft 2020-12.** In `jsonschema`, that's `Draft202012Validator`.
- **Pin the schema version you validate against.** Keep a copy of the schema alongside your code, or fetch it from `https://sirf2.org/schemas/sirf.schema.json` and store it, so a change to the published schema doesn't change your results unexpectedly.
