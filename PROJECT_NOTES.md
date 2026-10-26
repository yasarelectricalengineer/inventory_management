# Stockroom – Project Notes (what was built, and how)

**This copy: Version C = Version B + editable Google Drive folder link + several categories per product. See Version 8.** App version id: `2026.10.24-C` (shown in Settings > Storage).

Keep this file. If something breaks later, paste this file (or the relevant part) together with the problem so it can be fixed quickly.

## What it is
A frontend-only product inventory app (HTML + CSS + vanilla JavaScript). No backend, no database, no API, no login, no libraries. Everything is stored in the browser (IndexedDB, with localStorage as fallback). Works as a static site on GitHub Pages, Cloudflare Pages, Netlify.

## Files
| File | Purpose |
|---|---|
| `index.html` | Page structure, all sections, dialogs, icon sprite, Content-Security-Policy meta tag |
| `css/styles.css` | Layout, light/dark theme, responsive rules (table becomes cards under 760px) |
| `js/app.js` | All logic: storage, state, rendering, search/filter, forms, import/export |
| `README.md` | Run locally, storage, import/export, deploy steps |
| `PROJECT_NOTES.md` | This file |

## Pages
Dashboard, Products, Add Product (also used for edit and duplicate), Product Details (`#product/<id>`), Categories, Companies, Reports / Export & Import, Settings.

## Data model (stored in IndexedDB database `stockroom-db`)
- `products`: `{ id, name, categoryId, suppliers[], stock, minStock, image (JPEG data URL or null), createdAt, updatedAt }`
- `suppliers[]` item: `{ id, companyId, price, code (optional), notes (optional) }`
- `categories`: `{ id, name, createdAt }`
- `companies`: `{ id, name, createdAt }`
- Settings in localStorage key `stockroom:settings`: `currency` (default `AED`), `seeded`, `lastBackup`, plus branding (`appName`, `logo`, `logoSize`, `colors`; see Version 6).
- Old format (one `companyId` + `price` per product) is converted to one supplier automatically on first load (`migrateProducts`), and old backups still import (`parseBackup`).

## Features, in the order they were built

### Version 1: core app
- Dashboard: total products, categories, companies, stock quantity, inventory value, low stock, out of stock.
- Stock status: Out of Stock when stock = 0; Low Stock when stock > 0 and <= minimum; otherwise In Stock.
- Products CRUD with image (picked from device, resized to max 640 px, saved as JPEG, preview, change, remove).
- Categories and Companies: add, rename, delete, search. A category or company in use cannot be deleted. Names must be unique.
- Fire Alarm and Fire Fighting categories are created once on first run.
- Backup: Export JSON, Import JSON (replaces all data after confirmation), Export CSV, Clear All Data (confirmation dialog).
- Responsive layout with sidebar, off-canvas menu on mobile, toast messages, dark mode follows the system.
- Privacy: CSP meta tag blocks network connections; text is HTML-escaped; CSV cells are protected against formula injection; imported images must be `data:image/...`.

### Version 2: multi-supplier price comparison
- A product can have unlimited suppliers (company, price, optional code, optional notes). A company can appear once per product.
- Lowest price is calculated automatically (`priceInfo`), highlighted with a green **LOWEST PRICE** badge, live in the form and on the details page.
- Product Details page: Best Price, Supplier, saving message ("You can save ... compared with the highest price"), and the Supplier Price Comparison table (supplier, code, price, difference = price - lowest, status, notes).
- Product list columns: Lowest Price, Best Supplier (plus supplier count).
- Filters: category, supplier, status, min/max price, sort. Price range and price sorting use the lowest price, or the selected supplier's price when a supplier is chosen.
- Dashboard and reports value inventory as **stock x lowest supplier price**.
- Reports page: "Biggest supplier price differences" list.
- CSV: lowest price, best supplier, highest price, number of suppliers, all suppliers with prices and codes, stock, status, stock value.
- JSON backup version is now 2.

### Version 3: search and duplicate
- Large search box on the Products tab. Searches name, category, every supplier company, supplier code, supplier notes, any price and stock status. Several words can be typed and every word must match.
- Duplicate product: opens Add Product pre-filled as "<name> (Copy)" with suppliers and image; saved only when Save is clicked.
- Duplicate category and company: asks for a new name (suggests "<name> (Copy)").

### Version 4: manual Google Drive backup (no API)
- "Export to Google Drive": downloads the JSON backup, then opens the Drive folder in a new tab. The user uploads the file by hand.
- "Import from Google Drive": opens the Drive folder in a new tab and shows a step dialog; the user downloads the file, returns, clicks "Choose downloaded file", and the normal import runs.
- Folder used: `https://drive.google.com/drive/folders/1pCFabqhdlKb4VHPfEfAaBbRmhT5k1_RW` (constant `DRIVE_URL` near `driveExport` in `js/app.js`; change that one line to use another folder).
- No Google API, OAuth, Cloud project or billing is used anywhere.

### Version 5: bulk category edit
- Products table has a checkbox on each row and a select-all checkbox (selects only the products currently shown after search/filters).
- (Changed in Version 8: the bar now has Add category / Remove category / Remove all categories.) When products are selected a bar appears: pick a category and click **Set category** (replaced each product's single category), or click **Remove category** (cleared it). Both ask for confirmation, then save in one transaction (`Store.putMany`) and show a toast. Only products that actually change are counted.
- Selection is limited to visible products: if a filter hides a selected product it is unselected, so bulk actions never touch hidden rows.
- Category is now **optional** on a product (needed so "remove" makes sense). Products without one show "No category"; the Category filter has a "No category" option to find them.
- Code (Version 5 names; see Version 8 for the current ones): `updateSelectionUI`, `bulkApplyCategory`, `bulkSetCategory`, `bulkRemoveCategory`.

### Version 6: editable branding and fire theme (requested 2026-10-08)
- Settings has a **Branding** panel: app name (default `Ameer Fire & Safety`, max 40 chars, empty = default), logo upload, logo size slider (16-64 px, default 24 px = the old icon size), color presets and three color pickers (main color, highlight color, menu background), and **Reset to default**.
- The logo replaces the small icon beside the name in the sidebar. It is converted to a PNG of at most 128 px (transparent background kept), saved in `localStorage` settings. The favicon uses the logo (or a box icon in the main color) and the browser tab title uses the app name.
- Theme: all blue was removed. Neutral black/gray surfaces; sidebar black `#0b0b0c`; main color fire red `#e11d2e`; highlight fire yellow `#ffc400`. Presets: Fire Red + Yellow (default), Fire Red, Fire Yellow. Text on buttons switches to black automatically when the main color is light (yellow) so it stays readable; link color is adjusted for contrast.
- Settings keys (localStorage `stockroom:settings`): `appName`, `logo` (PNG data URL or null), `logoSize`, `colors {main, accent, sidebar}`.
- Backup JSON `settings` now also holds `appName`, `logo`, `logoSize`, `colors`; Import restores them (a backup without these fields leaves the current branding unchanged). Clear All Data does **not** reset branding; use "Reset to default" for that.
- Code: `sanitizeBranding`, `applyBranding`, `logoToDataURL`, `renderBrandingPanel`, `DEFAULT_NAME`, `PRESETS` in `js/app.js`; `.brand*`, `.preset`, `.logo-preview` and the color variables at the top of `css/styles.css`.
- Also fixed: the search icon overlapping the text in the top-bar search box.
- Also fixed: on phone-width product cards the Lowest Price / Best Supplier values were spread across the row.
- Build id: `index.html` (meta `app-build` and the `?v=` on the css/js links) and `js/app.js` (`APP_BUILD`) carry the same id. The `?v=` stops browsers loading old cached files; if the ids differ (partial upload) the app shows a warning. The id is shown in Settings > Storage as "App version".

### Version 7 (Version B only): bulk selection fix (requested 2026-10-08)
- Report: only the header checkbox appeared, row checkboxes did not, so bulk edit seemed broken.
- Investigation: in a real Chromium test (desktop and phone width) every row had its checkbox and the whole flow (select, Set category, Remove category, select all) worked with no errors. The reported symptom matches a **new `index.html` (header checkbox) running with an old cached `js/app.js`** (which does not draw row checkboxes). So the main fix is the anti-cache protection described in Version 6 (`?v=` on the css/js links, plus the mixed-version warning) and the "App version" line in Settings. If it still happens, send a screenshot and the F12 Console text.
- Improvement in this version: a visible **Select all shown** checkbox above the table (the table header is hidden on phones, so there was no select-all there). It stays in sync with the header checkbox and shows a partial state when only some rows are ticked.

### Version 8 (this copy): editable Drive folder link and several categories per product (requested 2026-10-24)
Base: the uploaded `inventory_new-main.zip`, which was identical to Version B.

**Google Drive folder link**
- Settings > **Google Drive backup folder**: text box + Save link / Open folder / Use original link. Stored in `localStorage` settings as `driveUrl`. The old hard-coded `DRIVE_URL` constant is gone; `DEFAULT_DRIVE_URL` (your original folder) is only the starting value.
- `cleanDriveUrl()` accepts only `https://drive.google.com/...` links (no http, no other hosts, no `user:pass@`), or a bare folder id (converted to a folder link). Empty = opens `https://drive.google.com/drive/my-drive`.
- `openDrive()` uses `settings.driveUrl`. The link is **not** included in the JSON backup (so importing someone else's backup can never change where your Drive buttons point).
- Tip: because the link now lives in the browser, you can delete the folder id from `DEFAULT_DRIVE_URL` in `js/app.js` before publishing the code publicly, then paste your link in Settings.

**Several categories per product**
- Data: `product.categoryId` (one) became `product.categoryIds` (array). Existing data is converted at load by `migrateProducts()` (and the old field is removed from storage); `parseBackup()` accepts both old backups (`categoryId`) and new ones (`categoryIds`, unknown ids and duplicates dropped).
- Form: category chips (tick several) + "New category" which creates and ticks the new one. Code: `renderCategoryChips`, `state.formCategories`.
- List and details show category pills (`catPills`); search matches all category names; the Category filter matches products having that category; "No category" matches products with none.
- Bulk bar: **Add category** (adds to each selected product, keeps the others), **Remove category** (removes only the chosen one), **Remove all categories**. Code: `bulkApplyCategory(mode, id)` with mode `add` / `remove` / `clear`, `bulkAddCategory`, `bulkRemoveCategory`, `bulkRemoveAllCategories`. Each asks for confirmation.
- Dashboard "Inventory value by category" and the Reports category table count a product once in **each** of its categories (so the rows add up to more than the total); a note on both pages says so. Products with no category appear under "Uncategorized". The dashboard total inventory value is unchanged.
- The Categories page count and the "cannot delete a category that is in use" rule use `categoryIds`.
- CSV: column "Categories", several joined with "; ".

## Where to look in `js/app.js`
- `Store` – IndexedDB/localStorage layer (`getAll`, `put`, `remove`, `replaceAll`)
- `priceInfo`, `bestLabel`, `lowPrice` – price comparison maths
- `filteredProducts`, `effPrice` – search, filters, sorting
- `renderProducts`, `renderProductDetail`, `renderDashboard`, `renderReports` – screens
- `renderSupplierRows`, `updateLiveLowest`, `readAndValidateForm`, `submitProduct`, `loadForm` – product form
- `startEdit`, `startDuplicate`, `duplicateItem` – edit and duplicate
- `updateSelectionUI`, `bulkApplyCategory(mode, id)`, `bulkAddCategory`, `bulkRemoveCategory`, `bulkRemoveAllCategories` – bulk category edit
- `catIdsOf`, `catNames`, `catPills`, `renderCategoryChips` – several categories per product
- `cleanDriveUrl`, `DEFAULT_DRIVE_URL`, `openDrive` – Google Drive folder link
- `exportJSON`, `exportCSV`, `parseBackup`, `importFile`, `clearAll` – backup
- `driveExport`, `driveImport`, `DRIVE_URL` – Google Drive buttons
- `sanitizeBranding`, `applyBranding`, `logoToDataURL`, `renderBrandingPanel` – branding
- `migrateProducts`, `init` – startup

## Things to know (limits and decisions)
- Data lives only in the browser profile and address (`file://`, `localhost` and the hosted URL each have separate data). Clearing site data or using private mode loses it. Export backups regularly.
- Import replaces all current data; there is no merge.
- Browser storage is limited. Many large images can fill it; images are shrunk to reduce this.
- The Drive folder link is visible in the site's source code if the site is public. Keep the folder private (not "Anyone with the link"), so only the signed-in owner can open it.
- Drive buttons only open a normal web page in a new tab; pop-up blockers can stop it (the app shows a message if so).

## Testing status (honest note)
- The JavaScript passes a syntax check, and a script checked the price maths (lowest 39 / highest 52 / saving 13), ties, stock status rules, CSV formula guard and import/migration/sanitising.
- The app was **not opened in a real browser** while building, so screen layout, clicks, dialogs, image upload and the Drive tab opening still need a manual check.

## Quick manual test checklist
1. Add 2 companies and a category; add a product with 3 suppliers (prices 45, 39, 52): the 39 row shows LOWEST PRICE live.
2. Save; open the product: best price, supplier, differences (+6, 0, +13) and "save 13" are correct.
3. Products tab: type part of a company name in the big search; filter by supplier and price range.
4. Duplicate a product, a category and a company.
5. Edit a price: lowest badge moves automatically.
6. Refresh and close/reopen the browser: data is still there.
7. Export JSON, Clear All Data, Import the same file: everything returns.
8. Export to Google Drive: file downloads and the Drive folder opens; Import from Google Drive: dialog shows and file picker opens.
9. Check on a phone-width window: menu, cards, forms.
11. Settings > Branding: change the name, upload a logo (it appears at 24 px by default), try the three color presets, then refresh: everything stays. Export a backup, Reset to default, Import the backup: branding returns.
10b. On a phone-width window tick "Select all shown": every card is ticked and the bulk bar shows the count (Version B only).
10. Tick 3 products, choose a category, click Add category: all 3 get it and keep their other categories. Choose the same category and click Remove category: only that one goes. Remove all categories: they show "No category"; filter by "No category" finds them.
13. Add a product and tick two categories: the list and details show both; filtering by either finds it; editing and unticking one removes it.
14. Settings > Google Drive backup folder: paste another folder link, Save; click Export to Google Drive: that folder opens. Try a non-Drive link: it is rejected.

## How to report a problem
Say: which page, what you clicked, what you expected, what happened, which browser, and any red error text from the browser console (press F12, then Console).
