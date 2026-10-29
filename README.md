# Inventory & Expiry Monitor

Single-page dashboard that reads a published Google Sheet (CSV) and classifies stock by expiry date. No backend, no build step.

## Deploy to GitHub Pages

1. Create a new GitHub repository (for example `inventory-expiry-dashboard`).
2. Upload everything in this folder to the repository root (`index.html` must be at the top level).
3. Go to **Settings > Pages**. Under **Build and deployment**, set **Source** to **Deploy from a branch**, branch **main**, folder **/ (root)**, then **Save**.
4. After a minute the site is live at `https://<your-username>.github.io/<repository-name>/`.
5. Open the site and check that the header says **Live from Google Sheets**.

## Data source

The CSV link is set once, near the top of the script in `index.html`:

```js
const SHEET_URL = 'https://docs.google.com/spreadsheets/d/e/.../pub?gid=0&single=true&output=csv';
```

- The sheet must stay published: **File > Share > Publish to web > CSV**.
- The page checks the sheet every 5 minutes (`REFRESH_MS`) and redraws only when the content changed. Google may cache a published sheet for a few minutes after an edit.
- If the link is unreachable, the page shows the saved copy of the sheet embedded in `index.html` and says so in the header.

## Column mapping

The header row is found automatically: the dashboard scores the first 60 rows and picks the one that best matches the fields below, so title rows or notes above it are ignored. Columns are matched by name; unknown columns are ignored.

| Dashboard field | Header names accepted |
|---|---|
| Product Name | Description, Product, Item Name |
| SKU | Material Number, SKU, Item Code |
| Batch Number | Batch, Batch No, Lot (the Storage Bin is shown if there is none) |
| Inventory Quantity | Total Count, Quantity, Qty, On Hand |
| Expiry Date | Expiry Date, Expiration Date, Exp Date |
| Unit Cost | Unit Cost, Cost, Cost per BX, Standard Cost, Unit Price, Price |
| Line total (used only if there is no unit cost) | Total Cost, Total Value, Amount |
| Category | Category, Brand (otherwise the brand is read from the product name) |

**Amount = Unit Cost x Total Count.** Lines with no unit cost show **N/A** and are left out of every value total. Dates are accepted as m/d/yyyy, yyyy-mm-dd, Excel serial numbers or text dates; rows with a missing or invalid expiry date are left out and counted in a notice on the page.

## Expiry rules

Days remaining = expiry date minus today (browser date).

| Status | Days remaining |
|---|---|
| Expired | below 0 |
| Critical (<30 Days) | 0 to 30 |
| Warning (<60 Days) | 31 to 60 |
| Monitor (<90 Days) | 61 to 90 |
| Safe | over 90 |

The `<30`, `<60` and `<90` toggles are cumulative and exclude expired stock.

## How the filters work

The expiry toggle (All Products, Expired, <30, <60, <90) is a global filter. Selecting one updates everything at once: the KPI cards, alert counts, value-at-risk row, all seven charts, the 60-day section, the main table, the four monitoring tables, totals, and every export.

- Search, product category and minimum amount narrow the same data further. Search matches product name, SKU, batch or bin, category, and the amount (for example `1395`).
- The numbers on the toggle buttons always show the count for each window (with search, category and minimum amount applied), so you can see what each button will show before you click it.
- With the Expired toggle on, the 60-day section includes expired lines automatically.

## Exports

**Export menu (All Products section)**
- Excel: entire dataset, filtered dataset, all four monitoring tables in one workbook.
- PDF: dashboard summary report, filtered report, inventory risk report.

**Elsewhere:** the 60-day section has its own Excel and PDF export, and each monitoring table has an Export Excel button.

All exports respect the active filters. Every export includes the company, site and count date from the sheet, export date and time, the filters applied, and totals and a status summary. Excel files have a Summary sheet first. PDFs write amounts as `PHP 1,234.00` because PDF built-in fonts have no peso sign.

## Tags and alerts

- **High value:** an expired, critical or warning line whose Amount is in the top quarter of all lines (needs at least 8 costed lines). Without any unit cost the dashboard tags **High qty** on critical lines instead.
- Expired rows and critical rows are shaded; high value rows are shaded darker.

## Libraries (loaded from cdnjs)

Chart.js 4.4.1, SheetJS 0.18.5 (Excel export), jsPDF 2.5.1 and jsPDF-AutoTable 3.5.31 (PDF export). Fonts from Google Fonts.
