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
- The page refreshes every 5 minutes (`REFRESH_MS`). Google may cache a published sheet for a few minutes after an edit.
- If the link is unreachable, the page shows the saved copy of the sheet embedded in `index.html` and says so in the header.

## Column mapping

The header row is found automatically (the first row containing an "expiry" column). Columns are matched by name:

| Dashboard field | Sheet column used now | Other names accepted |
|---|---|---|
| Product Name | Description: | Product, Item Name |
| SKU | Material Number: | SKU, Item Code |
| Batch Number | none (Storage Bin shown) | Batch, Batch No, Lot |
| Inventory Quantity | Total Count | Quantity, Qty, On Hand |
| Expiry Date | Expiry Date | Expiration Date, Exp Date |
| Unit Cost | none | Unit Cost, Cost, Unit Price |
| Category | none (brand from product name) | Category, Brand |

Add a column with one of these names and it is picked up on the next refresh.

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

## Libraries (loaded from cdnjs)

Chart.js 4.4.1, SheetJS 0.18.5 (Excel export), jsPDF 2.5.1 and jsPDF-AutoTable 3.5.31 (PDF export). Fonts from Google Fonts.
