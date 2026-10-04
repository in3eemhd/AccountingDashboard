# Architecture: Excel-fed SharePoint Accounting Dashboard

## Goal

A SharePoint page that renders the CEO Executive Financial Dashboard from a single
Excel workbook. One person (initially the CEO, later a delegate) uploads a new
version of the workbook; the dashboard reflects it on the next page load with no
manual step in between.

## Recommended design: SPFx web parts reading the workbook through Microsoft Graph

```
┌────────────────────────────┐    upload / overwrite    ┌─────────────────────────────┐
│ Finance owner              │ ───────────────────────▶ │ SharePoint document library │
│ (upload web part or drag   │                          │ "Dashboard Data"            │
│  and drop into library)    │                          │  CEO_Executive_Financial_   │
└────────────────────────────┘                          │  Dashboard.xlsx (versioned) │
                                                        └──────────────┬──────────────┘
                                                                       │ Graph Excel API
                                                                       │ /drive/items/{id}/workbook/
                                                                       │   worksheets/{name}/usedRange
                                                                       ▼
┌────────────────────────────┐   renders on page load   ┌─────────────────────────────┐
│ Site visitors (read-only)  │ ◀─────────────────────── │ SPFx dashboard web part     │
│                            │                          │ (React, Fluent UI, charts)  │
└────────────────────────────┘                          └─────────────────────────────┘
```

### Why this option

| Option | Freshness | Licensing | Effort | Verdict |
|---|---|---|---|---|
| **SPFx web part + Graph Excel API (recommended)** | Immediate: the web part reads the current file every load (cached a few minutes) | None beyond M365 | Medium | Chosen |
| Power BI report embedded in SharePoint | Scheduled refresh only (8/day on Pro), every viewer needs a Pro license | Power BI Pro per user | Medium | Rejected: not immediate, licence cost |
| Power Automate flow copies Excel rows into SharePoint lists; web part reads lists | 1 to 5 minutes after upload | None | Medium-high | Fallback if Graph permissions cannot be approved |
| Excel Online "File viewer" web part | Immediate | None | Low | Rejected: shows a spreadsheet, not a dashboard |

### How "immediate" works

1. The workbook lives in a document library at a fixed path. Versioning is on, so
   every upload keeps history and can be rolled back.
2. The dashboard web part calls Graph for the file's `lastModifiedDateTime` and the
   used range of each sheet (`CEO Dashboard`, `Accounts Receivable`,
   `Accounts Payable`, `Loan Schedule`, `Cash Position`). Cached values are returned,
   so Excel formulas never have to be re-evaluated in the browser.
3. Responses are cached in session storage keyed by the file's eTag. A new upload
   changes the eTag, so the next page load fetches fresh data. The header shows
   "Data as of {lastModifiedDateTime} by {modifiedBy}".
4. No flows, no scheduled jobs, no admin step per upload.

### Permissions and the upload section

* The library gets a unique permission set: a "Dashboard Data Editors" SharePoint
  group with Contribute, everyone else Read. Moving upload rights to someone else
  is a group-membership change, nothing in code.
* The upload web part (shown in the mockup as the "Data source" panel) writes to
  that library using the current user's permissions. Users without Contribute see
  the panel in read-only state (file name, last updated, uploaded by, version).
* Graph scopes required: `Sites.Read.All` (delegated) or `Files.Read.All`. These
  are requested once in `package-solution.json` and approved by a tenant admin in
  the SharePoint admin center API access page.

### Workbook contract

The web part depends on sheet names and header labels, not on cell addresses.
Each sheet is read as a table with a header row. Renaming a sheet or a column
header is the only change that would break the dashboard; the web part reports
exactly which sheet or column it could not find rather than failing silently.

## Repository layout (planned)

```
design/      HTML mockups for review (this phase)
docs/        architecture and decisions
spfx/        SharePoint Framework solution (next phase)
```
