# Standard Industry Reporting Format (SIRF) — Delivery

SIRF data can be delivered in two ways:

| Method | What is sent | Used for |
| --- | --- | --- |
| [JSON Lines file](#json-lines-files) | One Header record followed by many Account records | Bulk reporting of a whole portfolio for a reporting period |
| [HTTP POST](#single-records-over-http) | One Account record | Real-time reporting of a single account, or an individual correction |

Both methods use the same Account record. The one difference is `portfolioId`: in a file it comes from the Header record, while a record sent on its own carries it in its `account` object. For the fields themselves, see the [Format Reference](sirf-schema.md). For how to check records before sending them, see [Validation](sirf-validation.md).

---

## JSON Lines Files

### What JSON Lines is

[JSON Lines](https://jsonlines.org/) is a simple text format in which **every line is a complete JSON object**. A file is not one large JSON document wrapped in `[ … ]`: each line stands on its own and can be read, parsed and validated independently.

The JSON Lines format itself sets three rules:

- The file is encoded as **UTF-8**.
- Each line is a single, valid JSON value. For SIRF, that's always a JSON object.
- Lines are separated by a newline character (`\n`).

The conventional file extension is `.jsonl`.

### Why SIRF uses it

- **Large files are easy to process.** A receiver can read one line at a time, so a file with millions of accounts never has to be loaded into memory all at once.
- **Errors point to a line.** A problem in one record can be reported by its line number, such as "line 4: `daysPastDue` is required", rather than as a failure of the whole file.
- **Files are easy to produce.** A lender can write a file record by record, straight from a database query or export job, with no need to build one large document first.
- **Files can be split and processed in parallel.** Because no record depends on the lines around it, a large file can be divided between several workers.
- **Standard tools work.** Line-based tools such as `wc -l`, `head` and `grep` work on SIRF files directly.

### File structure

Every SIRF file has the same shape:

```text
Line 1:  Header record   — format version, reporting period, and portfolio metadata
Line 2+: Account record  — one consumer credit account per line
```

- **Line 1 is always the Header record.** It carries `sirfVersion`, the reporting period (`reportingStartDate` and `reportingEndDate`), the `portfolioId`, and the `recordCount`. This context applies to every account record in the file.
- **Every other line is an Account record.** Each one holds a single account: `recordType`, `person`, `address` and `account`. Account records in a file don't include `account.portfolioId`, because the header's `portfolioId` applies to them all.
- **`recordCount` must equal the number of account records** in the file, not counting the header. It's at least 1, so a file always contains at least one account.
- **Each record sits on a single line.** Records are written in compact form, with no line breaks inside a record and no array wrapper around the file. Pretty-printed records, like the examples in the Format Reference, are only for reading.

### Example file

A complete SIRF file with a header and five account records. This is [samples/sample-batch.jsonl](samples/sample-batch.jsonl):

```json
{"recordType":"Header","sirfVersion":"2.0","reportingStartDate":"2025-02-01","reportingEndDate":"2025-02-28","portfolioId":"farringdon-mortgages","recordCount":5}
{"recordType":"Account","person":{"customerId":"FAR-C10482","title":"Mrs","firstName":"Patricia","lastName":"Okafor","dob":"1968-07-30"},"address":{"buildingName":"Rose Cottage","buildingNumber":"8","line1":"Church Lane","line2":"Clifton","city":"Bristol","postalCode":"BS8 4CD"},"account":{"accountId":"POK-7714","accountType":"Mortgage","accountSubtype":"Residential","status":"UpToDate","startDate":"2018-03-01","closeDate":"2025-03-01","repayment":499,"repaymentPeriod":300,"currentBalance":87450,"paymentFrequency":"Monthly"}}
{"recordType":"Account","person":{"customerId":"FAR-C20931","title":"Mr","firstName":"James","lastName":"Thornton","dob":"1979-11-14","email":"j.thornton@example.com","phone":"447700900142"},"address":{"buildingNumber":"42","line1":"Maple Street","city":"Manchester","postalCode":"M1 4BT"},"account":{"accountId":"JTH-2293","accountType":"CreditCard","status":"UpToDate","startDate":"2021-09-15","repayment":75,"repaymentPeriod":60,"currentBalance":1240,"paymentFrequency":"Monthly","creditLimit":5000,"minimumPayment":25,"cashAdvances":200,"cashAdvancesCount":2}}
{"recordType":"Account","person":{"customerId":"FAR-C31057","title":"Ms","firstName":"Sarah","middleName":"Louise","lastName":"Mitchell","dob":"1985-03-22"},"address":{"buildingNumber":"14","line1":"Birchwood Avenue","line2":"Headingley","city":"Leeds","postalCode":"LS6 2AB"},"account":{"accountId":"SML-4821","accountType":"UnsecuredLoan","status":"Delinquent2","startDate":"2023-06-01","daysPastDue":62,"repayment":185,"repaymentPeriod":36,"startBalance":6660,"currentBalance":3421,"paymentFrequency":"Monthly","flags":["Arrangement"]}}
{"recordType":"Account","person":{"title":"Mr","firstName":"David","lastName":"Nkosi","dob":"1995-06-12"},"address":{"buildingNumber":"31","line1":"Wellington Street","city":"Sheffield","postalCode":"S1 4ER"},"account":{"accountId":"DNK-8856","accountType":"HirePurchase","status":"UpToDate","startDate":"2022-09-01","repayment":320,"repaymentPeriod":60,"currentBalance":12800,"paymentFrequency":"Monthly"}}
{"recordType":"Account","person":{"customerId":"FAR-C48820","title":"Ms","firstName":"Eleanor","lastName":"Griffiths","dob":"1961-03-28"},"address":{"buildingName":"Willowbank House","buildingNumber":"5","line1":"Riverside Way","city":"Exeter","postalCode":"EX2 4AB"},"account":{"accountId":"EGR-1147","accountType":"Mortgage","accountSubtype":"BuyToLet","status":"Defaulted","startDate":"2015-06-01","closeDate":"2024-11-30","repayment":612,"repaymentPeriod":240,"currentBalance":0,"paymentFrequency":"Monthly","flags":["Partial"]}}
```

---

## Single Records over HTTP

A single Account record can also be sent on its own, as the body of an HTTP `POST` request. This supports two uses:

- **Real-time reporting.** An account can be reported as soon as something changes, such as a new account being opened or a payment being missed, without waiting for the next bulk file.
- **Individual corrections.** If one account was reported wrongly, the corrected record can be sent on its own, without re-sending a whole file.

### What is sent

The request body is **one Account record**, validated against the same schema definition as a line of a file, `$defs/accountRecord`. There is no Header record: a POST carries a single account only.

Without a header, the record identifies its portfolio itself. Include `portfolioId` in the `account` object, next to `accountId`, using the same format as the header's `portfolioId`. This is the only time `account.portfolioId` is used.

Because each record is self-contained JSON, the body can be pretty-printed or compact; both are the same JSON document.

### Example: real-time report

A credit card account reported as soon as it changes. This is [samples/sample-instance-credit-card.json](samples/sample-instance-credit-card.json):

```text
POST /report
Content-Type: application/json
```

```json
{
  "recordType": "Account",
  "person": {
    "customerId": "FAR-C55213",
    "title": "Mr",
    "firstName": "Marcus",
    "lastName": "Bell",
    "dob": "1983-12-07",
    "email": "m.bell@example.co.uk",
    "phone": "447911234567"
  },
  "address": {
    "buildingNumber": "9",
    "line1": "Fernside Close",
    "city": "Norwich",
    "postalCode": "NR3 2HT"
  },
  "account": {
    "accountId": "MBL-7722",
    "portfolioId": "farringdon-credit-cards",
    "accountType": "CreditCard",
    "status": "UpToDate",
    "startDate": "2020-11-01",
    "repayment": 150,
    "repaymentPeriod": 60,
    "currentBalance": 2340,
    "paymentFrequency": "Monthly",
    "creditLimit": 7500,
    "minimumPayment": 47,
    "cashAdvances": 300,
    "cashAdvancesCount": 1
  }
}
```

### Example: individual correction

Suppose the mortgage `POK-7714` was reported with the wrong current balance. The correction is the **complete** account record with the corrected value, identified by the same `accountId` and `portfolioId`:

```text
POST /report
Content-Type: application/json
```

```json
{
  "recordType": "Account",
  "person": {
    "customerId": "FAR-C10482",
    "title": "Mrs",
    "firstName": "Patricia",
    "lastName": "Okafor",
    "dob": "1968-07-30"
  },
  "address": {
    "buildingName": "Rose Cottage",
    "buildingNumber": "8",
    "line1": "Church Lane",
    "line2": "Clifton",
    "city": "Bristol",
    "postalCode": "BS8 4CD"
  },
  "account": {
    "accountId": "POK-7714",
    "portfolioId": "farringdon-mortgages",
    "accountType": "Mortgage",
    "accountSubtype": "Residential",
    "status": "UpToDate",
    "startDate": "2018-03-01",
    "closeDate": "2025-03-01",
    "repayment": 499,
    "repaymentPeriod": 300,
    "currentBalance": 86950,
    "paymentFrequency": "Monthly"
  }
}
```

A correction is a whole record, not just the changed field: the schema requires `person`, `address` and the account's core fields on every record, so a partial update would fail validation.

If the account's identifier itself has changed, send the record with the original `accountId` and the new identifier in `accountIdChange`. Processors replace the previous `accountId` with the `accountIdChange` value.
