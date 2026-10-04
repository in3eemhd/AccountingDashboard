# Workbook contract

The dashboard reads these sheets from `CEO_Executive_Financial_Dashboard.xlsx` by **sheet name and header label**. Cell addresses are never used, so rows can be added freely. Hidden sheets are read the same way as visible ones.

| Sheet | Read as | Required headers (row 4 unless noted) |
|---|---|---|
| `Dash Calc` | Key metric table A4:B26, aging table D4:G10, monthly trend I4:L18, loan runway N4:R17, department D14:F21, obligation type D25:G31, top 5 debtors/creditors I22:M37, KRIs O22:T28, facilities D35:G38 | `Key Metric`, `Value`, `Aging Bucket`, `AR`, `Operating AP`, `Month`, `Salaries Facility`, `Supplier Loans`, `Total Due`, `Remaining After Payment`, `Department`, `Amount`, `% of AR`, `Obligation Type`, `Indicator`, `Actual`, `Green limit`, `Amber limit`, `Better when`, `Status`, `Facility`, `Limit`, `OS Principal`, `Utilisation %` |
| `AR Data` | Open invoice list | `Invoice`, `Invoice Date`, `Client`, `Department`, `Amount Due (SAR)`, `Age (days)`, `Aging Bucket` |
| `AP Data` | Open item list | `Invoice`, `Invoice Date`, `Beneficiary`, `Account Name`, `Amount Due (SAR)`, `Age (days)`, `Aging Bucket`, `Group` |
| `Loan Schedule` | Instalment list | `Loan ID`, `Facility`, `Description`, `Inst. #`, `Tawarruq Ref`, `Principal`, `Interest`, `Due Date`, `Installment`, `Paid Amount`, `Balance`, `Status`, `OS Principal`, `Due Month` |
| `Cash Position` | Balance date B4, previous-day total B5, accounts A8:F13 (header row 7) | `Bank Account`, `Currency`, `Amount (CCY)`, `Rate to SAR`, `Amount (SAR)`, `% of cash` |
| `Executive Dashboard` (hidden) | Report date C4; liquidity outlook C18:F25; integrity checks B81:C88 | `Report date` |

## Notes found while mapping the workbook

* The report date lives on a hidden sheet (`Executive Dashboard!C4`). The dashboard shows it in the masthead so a stale date is visible.
* `Dash Calc!B21` ("30-day obligations: AP 0-30 + loans due this month") sums the *net* 0-30 bucket and only the salaries facility. The label-consistent figure would be AP 0-30 plus all October instalments. The mockup shows the workbook's own value (0.03x); worth confirming which definition the CEO wants before build.
* The AP bank-loan plug line exceeds the Loan Schedule remaining balance by 62,213 (one L534 instalment now marked Paid). The workbook flags this itself; the dashboard surfaces it under "Needs attention".
* Heritage receivables are listed under project/payment descriptions rather than a client name, so the top-5 debtors are effectively one client. The mockup notes this under the table.
