# stl-efile

Schema specifications for bulk electronic filing of St. Louis Earnings and Payroll Expense taxes with the City of St. Louis Collector of Revenue (STLCOR).

For questions about bulk electronic filing, please contact CORIT@stlouis-mo.gov.

## File processing, errors, and warnings

### Validation

Upon submitting a file, STLCOR will first validate the file against the schema. Invalid files will not be processed and a critical error will be reported to the submitter, along with all available validation errors.

### Initial processing

STLCOR will attempt to process valid files. The following problems will cause critical errors and prevent the file from being processed:

- Unregistered electronic filer (all electronic filers must register with STLCOR)
- Incorrectly calculated totals in the `<BatchHeader>` element:
    - `<TotalItems>` does not equal the total number of `<STLW10>`, `<STLP10>`, and/or `<STLW11>` elements in the file
    - `<AmountDueTotal>` does not equal the sum of all `<AmountDue>` elements within the `<STLW10>`, `<STLP10>`, and/or `<STLW11>` elements
    - `<RemittanceTotal>` does not equal the sum of all `<Remittance>` elements within the `<STLW10>`, `<STLP10>`, and/or `<STLW11>` elements
- Exact duplicate files submitted

Any of these critical errors will be reported back to the submitter. The entire file will be rejected and no returns will be applied to taxpayer accounts.

### Final processing

If no critical errors are encountered, STLCOR will complete processing the file and apply the returns to taxpayer accounts as specified in the file.

Exceptions that do not prevent processing the file will be reported back to the submitter. These exception reports are for information only and do not require immediate resolution or resubmission by the submitter.

#### Possible exceptions

- **Entity not on file:** the taxpayer is not registered with STLCOR. STLCOR will treat this as a new entity and register the entity on the fly.
- **Account type not on file:** the specified return type does not match the taxpayer's active accounts. For example, if a P-10 return is filed for a taxpayer that previously only filed W-10 returns, then an exception would be reported. STLCOR will open the corresponding account type to accept the filing.
- **Gross tax due calculated incorrectly:** the `<GrossTaxDue>` element contains a value that is not exactly 1 percent of the `<TaxableEarnings>` or `<TaxablePayroll>` reported. STLCOR will adjust `<GrossTaxDue>` accordingly. The recalculation may result in a change to the `<AmountDue>` and cause a balance due or a credit on the taxpayer's account.
- **Net tax due calculated incorrectly:** the `<NetTaxDue>` element contains a value that is not exactly 1 percent of the `<TaxableEarnings>` or `<TaxablePayroll>` reported minus any `<PriorPayments>` on file with STLCOR. For example, if the submitter reports a previous deposit that does not match the actual amount received by STLCOR, the `<PriorPayments>` value will be adjusted to match the true amount deposited and recalculate `<NetTaxDue>` accordingly. The recalculation may result in a change to the `<AmountDue>` and cause a balance due or a credit on the taxpayer's account.
- **Return already on file:** if a non-amended W-10 or P-10 is submitted for an account and period which already has a non-amended W-10 or P-10 on file, STLCOR will report this to the submitter. The remedy for this exception varies situationally; STLCOR may reach out to the submitter or the taxpayer for clarification.
- **Duplicate return:** similarly to the return already on file exception, the submitter will be notified if a return is an exact match of one already on file for the account and period. Again, the remedy for this exception varies situationally; STLCOR may reach out to the submitter or the taxpayer for clarification.

## Remittance

Upon successful processing, STLCOR will issue an invoice to the submitter for the amount reported in `<RemittanceTotal>`. The submitter will send a single ACH Credit transaction to STLCOR. When the funds are received, STLCOR will apply the payments as indicated in the `<Remittance>` element of each `<STLW10>`, `<STLP10>`, and/or `<STLW11>`.

Note that if exceptions were reported and adjustments were made due to miscalculations, the remittances will be applied to the recalculated values and applied in accordance with STLCOR policies.

### Example 1: Gross tax due calculated incorrectly

The following miscalculation scenario will cause the taxpayer to have a balance due of $25.00.

|                   | Reported by submitter | Calculated by STLCOR | Applied by STLCOR |
| ----------------- | --------------------: | -------------------: | ----------------: |
| `TaxableEarnings` | $100,000.00           | $100,000.00          | $100,000.00       |
| `NetTaxDue`       | $975.00               | $1,000.00            | $1,000.00         |
| `AmountDue`       | $975.00               | $1,000.00            | $1,000.00         |
| `Remittance`      | $975.00               | $975.00              | $975.00           |

### Example 2: Net tax due calculated incorrectly

In this scenario, the `PriorPayments` reported by the submitter do not match those on file with STLCOR. The shortage will cause the taxpayer to have a balance due of $300.00.

|                   | Reported by submitter | Calculated by STLCOR | Applied by STLCOR |
| ----------------- | --------------------: | -------------------: | ----------------: |
| `TaxableEarnings` | $100,000.00           | $100,000.00          | $100,000.00       |
| `GrossTaxDue`     | $1,000.00             | $1,000.00            | $1,000.00         |
| `PriorPayments`   | $600.00               | $300.00              | $300.00           |
| `NetTaxDue`       | $400.00               | $700.00              | $700.00           |
| `AmountDue`       | $400.00               | $700.00              | $700.00           |
| `Remittance`      | $400.00               | $400.00              | $400.00           |

### Example 3: Penalty and interest calculated incorrectly

Although the return was due on April 30th, the submitter sent the taxpayer's return on June 5th. Unfortunately, the submitter calculated the penalty and interest due incorrectly.

|                   | Reported by submitter | Calculated by STLCOR | Applied by STLCOR |
| ----------------- | --------------------: | -------------------: | ----------------: |
| `TaxableEarnings` | $100,000.00           | $100,000.00          | $100,000.00       |
| `GrossTaxDue`     | $1,000.00             | $1,000.00            | $1,000.00         |
| `NetTaxDue`       | $1,000.00             | $1,000.00            | $1,000.00         |
| `PenaltyDue`      | $50.00                | $100.00              | $100.00           |
| `InterestDue`     | $10.00                | $20.00               | $20.00            |
| `AmountDue`       | $1,060.00             | $1,120.00            | $1,120.00         |
| `Remittance`      | $1,060.00             | $1,060.00            | $1,060.00         |

STLCOR applies payments in the following order:

1. Fees due
2. Penalty due
3. Interest due
4. Tax due

As such, the $1,060.00 payment will be applied as follows and the taxpayer will have a remaining balance due of $60.00.

|          | Balance due | Balance paid | Remaining balance |
| -------- | ----------: | -----------: | ----------------: |
| Tax      |   $1,000.00 |      $940.00 |            $60.00 |
| Penalty  |     $100.00 |      $100.00 |                   |
| Interest |      $20.00 |       $20.00 |                   |
| Total    |   $1,120.00 |    $1,060.00 |            $60.00 |

## Response and Invoice

Upon receipt of a file, STLCOR will validate the file and perform initial processing. STLCOR will make a response file available with a status of `ACCEPTED_PENDING` or `REJECTED`. If the file is rejected, the response file will include an itemized list of errors.

STLCOR will continue processing the file, logging exceptions, and posting returns to taxpayer accounts. Upon completion of processing, STLCOR will provide an invoice.