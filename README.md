# SEDAR+ Annual MD&A Filings: Visualization

A 3-page Power BI report that explores a SEDAR+ export of Annual MD&A filings by Canadian province, language, and company.

> **Status:** exploratory draft. Several visuals use measures that don't mean anything for this data (see [Known issues](#known-issues-and-recommended-fixes)). The reliable findings are listed under [What the data supports](#what-the-data-supports).

---

## Purpose

Answer basic questions about a batch of recently filed Annual MD&A documents (Management's Discussion and Analysis, the narrative that accompanies a company's annual financial statements):

- Which provinces do the filings come from?
- How many filings are in English vs. French?
- Which companies filed more than once?
- Where is the volume (and file size) concentrated?

## Data source

| Item | Detail |
|---|---|
| Source | SEDAR+ (sedarplus.ca), the Canadian securities filing system |
| Raw file | `export-26-10-19_12_01_PM.csv` (exported 2026-10-19) |
| Cleaned file | `sedar_annual_mdna_cleaned.xlsx`, sheet `Filings` |
| Filing type | Annual MD&A |
| Search window | 19 Oct 2021 to 19 Oct 2026 |

**Important:** SEDAR+ matched about 10,000 records but exported only the **1,800 newest**. The data therefore covers roughly **1 April 2026 to 19 October 2026**, not the full five-year window. Every number in this report describes that sample, not all Canadian filings.

## Data model

One table, one row per filing document.

| Column | Type | Description |
|---|---|---|
| Company | Text | Company name (English) |
| Alternate (French) Name | Text | French name, blank for 1,702 of 1,800 rows |
| SEDAR+ Profile No. | Text | Company identifier. An ID, not a quantity. Keep as text. |
| Document Type | Text | Always "Annual MD&A" in this extract |
| Language | Text | English (1,731) or French (69) |
| Submitted (Eastern Time) | Date/time | When the filing was submitted |
| Province | Text | Principal jurisdiction (8 provinces appear) |
| File Size (KB) | Whole number | Size of the PDF |

**Cleaning applied:** removed the search-settings lines and the truncation warning, split the original "Profile(s)" column into name and ID, split "Document" into type and language, converted the date text to a real date/time, and converted file size to a number. Details are on the `Notes` sheet of the workbook.

---

## Report pages

### Page 1: Province overview

| Visual | Fields | What it shows |
|---|---|---|
| Count of Language by Province (column) | Province, Count of Language | **Filings per province.** British Columbia is about 1,050, Ontario about 450, then Alberta and Québec. |
| Sum of Submitted by Province (area) | Province, Sum of Submitted | Not meaningful (see known issues) |
| Sum of SEDAR+ Profile No. by Province (column) | Province, Sum of Profile No. | Not meaningful |
| Sum of SEDAR+ Profile No. by Company (line) | Company, Sum of Profile No. | Not meaningful |

### Page 2: Company and file size

| Visual | Fields | What it shows |
|---|---|---|
| Count of Language by Company (column, scrollable) | Company, Count of Language | **Filings per company.** Glen Eagle Resources Inc. has 3; many companies have 2; the long tail has 1. |
| Sum of File Size (KB) by Province (bar) | Province, Sum of File Size | **Total PDF volume per province.** BC is about 0.7M KB, Ontario about 0.3M KB. Tracks filing count. |

### Page 3: Language and detail

| Visual | Fields | What it shows |
|---|---|---|
| Count of Alternate (French) Name by Province (column) | Province, Count of Alternate Name | **Where French-named companies are.** Québec has about 60, BC about 21, Ontario about 11. |
| Count of Language by Province (area) | Province, Count of Language | Same data as the Page 1 column chart, drawn as an area |
| Count of Province by SEDAR+ Profile No. (line) | Profile No., Count of Province | Not meaningful |
| Sum of Submitted by Province (bar) | Province, Sum of Submitted | Not meaningful |
| Sum of SEDAR+ Profile No. by Language (column) | Language, Sum of Profile No. | Not meaningful |
| Count of Document Type by Language (100% area) | Language, Count of Document Type | Shows a flat 100% because there is only one document type |

---

## What the data supports

These findings are reliable for the 1,800-row sample:

- **BC dominates.** About 1,048 of 1,800 filings (58%) come from British Columbia, followed by Ontario (446), Alberta (150), and Québec (112). The remaining four provinces total 44.
- **Mostly English.** 1,731 English vs. 69 French documents (about 4% French).
- **French names cluster in Québec.** About 60 of the 98 companies with a French name are in Québec.
- **Most companies filed once.** 1,746 distinct companies across 1,800 filings. Only a few filed 2 or 3 documents, usually because they filed both an English and a French version.
- **File size follows volume.** Provinces with more filings have more total KB, so size isn't revealing anything beyond count.

## Known issues and recommended fixes

| Visual | Problem | Fix |
|---|---|---|
| Sum of Submitted by Province (2 visuals) | Adding up dates produces a meaningless number (about 48M for BC). | Replace with **Count of Profile No.** by month: Submitted on the axis, Count as the value. This shows filing activity over time. |
| Sum of SEDAR+ Profile No. (4 visuals) | A profile number is an ID. Summing IDs measures nothing. The totals just mirror how many filings each province has. | Change the aggregation to **Count (Distinct)** to show the number of companies, or remove the visuals. Set the column's **Summarization** to "Don't summarize" in the model. |
| Count of Province by SEDAR+ Profile No. (line) | Uses the ID as a continuous axis, so the spikes are ID ranges. | Remove. |
| Count of Document Type by Language (100% area) | Only one document type exists, so the chart is always flat at 100%. | Remove, or replace with a language split by province (100% stacked bar). |
| Count of Language by Province appears twice | Duplicate of the Page 1 column chart in a different form. | Keep the column chart, delete the area version. |
| Area/line charts on a category axis | Provinces have no order, so a line or area implies a trend that isn't there. | Use bar or column charts for every province comparison. |
| Titles read "Count of Language" | Counts rows but is labelled as if it were about language. | Create a measure `Filings = COUNTROWS(Filings)` and rename titles to "Filings by Province". |
| Truncated export | Only 1,800 of about 10,000 records are included. | Re-export with narrower filters (by province or date range) and combine the files. |

### Suggested measures

```dax
Filings = COUNTROWS(Filings)

Companies = DISTINCTCOUNT(Filings[SEDAR+ Profile No.])

French Share % = DIVIDE(CALCULATE([Filings], Filings[Language] = "French"), [Filings])

Avg File Size (KB) = AVERAGE(Filings[File Size (KB)])
```

---

## How to refresh or rebuild

1. In SEDAR+, run a search for **Annual MD&A**, narrow the filters so the result is under 1,800 rows (SEDAR+ truncates larger exports), and export to CSV.
2. Skip the first 10 search-settings lines. The table header begins at the row starting with `Profile(s)`.
3. Split `Profile(s)` into company name and profile number, and `Document` into type and language. Convert the date and file size columns. (The cleaned workbook shows the target layout.)
4. In Power BI: **Home > Get Data > Excel**, select the `Filings` sheet, and confirm types (Profile No. as Text, Submitted as Date/Time, File Size as Whole Number).
5. Rebuild or refresh the visuals using the measures above.

## Limitations

- Sample only: the newest 1,800 of about 10,000 matching records.
- The search filters (amended statements, non-reporting issuers) may shape which companies appear, so the results may not represent all Canadian filers.
- The report shows filings, not company size, financial results, or filing quality.
- Province is the principal jurisdiction in SEDAR+, which isn't always where a company operates.

