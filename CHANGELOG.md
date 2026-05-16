# CHANGELOG — Able Perfect Price Calculator

---

## v2026-05-16 — Joel's Amendments Batch 1

### 1. USD/MT Toggle (YES/NO) now propagates to ALL outputs
- Quotation cart cards: FOB/MT and CIF/MT rows are hidden when toggle is set to NO
- `openPrintPreview()` HTML card view: Price/MT row hidden when NO
- `copyAsImage()` canvas (Image button): Price/MT row hidden, card height auto-adjusted
- `saveAsPDF()` canvas (PDF button): Price/MT row hidden, card height auto-adjusted
- `buildRpt()` WhatsApp text report: `/MT` omitted from price line when NO
- `generatePDF()` table PDF: was already correct via `showMT` flag (no change needed)

### 2. Quotation cart now reflects selected pricing option only
- Previously: FOB price was always shown in every cart item, even for CIF-only items
- Now: Each cart card shows ONLY the pricing block(s) relevant to that item's locked `priceMode`
  - `priceMode = 'fob'` → FOB pricing block only
  - `priceMode = 'cif'` → CIF pricing block only
  - `priceMode = 'both'` → FOB + CIF pricing blocks

### 3. FOB / CIF / FOB+CIF strictly segregated across all reports
- Each item locks its `priceMode` the moment "Add to Quote" is clicked
- Cart cards, WhatsApp text, table view, PDF, and image ALL render each item from its own locked `priceMode`
- Mode badge is colour-coded: 🟢 green = FOB, 🔵 blue = CIF, 🟣 purple = FOB+CIF
- Grand total bar in cart shows FOB total, CIF total, or both, depending on what items are in the cart

### 4. FCL quantity locked once quoted — "once click quoted, it is final"
- FCL input field removed from quotation cart items; replaced with a static read-only display
- Label changed to "FCL (locked)" to make it clear the quantity is committed
- Remove button remains available so items can be deleted if needed
- Max 50 items per quotation still enforced

---

## v2026-05-16b — Joel's Amendments Batch 2

### 5. Additive Type dropdown — new column in Step 3
- New dropdown placed beside the Additive (USD/MT) field in Step 3
- Full list of options:
  - TBHQ, BHA + TBHQ, BHA, Vit A, Vit D, Vit A + Vit D
  - Anti Foaming, Anti Clouding, Vit A + Antioxidant
  - Vit A + Antioxidant + Anti Foaming
  - Vit A + Anti Clouding
  - Vit A + Antioxidant + Anti Foaming + Anti Clouding
- Selection is locked per item the moment "Add to Quote" is clicked (`item.additiveType`)
- Each quotation item is fully independent — changing the dropdown after quoting does not affect already-quoted items
- Appears as `[Additive Type]` suffix appended to the product name in the Product & SKU column across all outputs:
  - Quotation cart item title
  - WhatsApp text report (Packaging line)
  - Table view (SKU Name column)
  - Text table (SKU Name column)
  - Mobile card view (product header)
  - Full modal card view
  - openPrintPreview() HTML cards
  - copyAsImage() canvas header
  - saveAsPDF() canvas header
  - generatePDF() PDF table (Product & SKU column)

### 6. Summary labels renamed
- "GRAND FOB" / "Grand Total FOB" → **Total FOB** everywhere
- "GRAND CIF" / "Grand Total CIF" → **Total CIF** everywhere
- "See above" in PDF table footer replaced with a calculated value

### 7. Estimated Grand Total of Sales
- When both FOB and CIF items are present in the quotation, all summary sections now show: **ESTIMATED GRAND TOTAL OF SALES = Total FOB + Total CIF**
- Displayed in gold/yellow to distinguish from individual totals
- Appears in: quotation cart bar, mobile table summary, renderVTable, generatePdfHtml, openPrintPreview summary card, canvas Image summary, canvas PDF summary, PDF table footer, buildRpt WhatsApp SUMMARY section

---


- FOB / CIF / Both pricing modes — each item locks its mode when added to quote
- Save PDF opens a new tab → user taps "Save as PDF" button → Chrome print dialog
- Price/MT column toggle (YES/NO) affects PDF layout
- Each item stores its own oil price, additive, margin, port independently
- Image export (copyAsImage) via Canvas API for WhatsApp sharing
- 634 ports across all regions with freshness indicators
- Password protection via session storage

## v2026-05-16c — Joel's Amendments Batch 3

### 8. Packaging costs updated (source: ABLE_UPDATED_EXHIBITION_CALCULATOR_2026_V1May13.xlsx)
- All 96 existing SKUs updated with new packaging cost/MT values from the Excel file
  - Fuzzy name matching used to handle minor spacing differences (e.g. "6 x 2kg" vs "6 x 2 kg")
  - Existing SKU names preserved exactly as-is to avoid breaking changes
- 8 genuinely new SKUs added:
  - **SQ TINS**: 17kg SM tin, 20L SM tin (Gross 17kgs), 15kg SM tin, 15kg SM tin w CTN, 15kg SM tin w CTN (Palletise @ 40'ft)
  - **FLEXIBAGS**: Flexible bag (21.5MT), Flexible bag + HP (21.5MT)
  - **BIB**: 15kg BAG IN BOX
- Total SKU count: **104** (up from 96)

### 9. Freight costs updated (source: ABLE_UPDATED_EXHIBITION_CALCULATOR_2026_V1May13.xlsx)
- 36 ports updated with new freight rates, including:
  - Africa: LOBITO, NAMIBE, DOUALA, MATADI, POINTE NOIRE, BATA (CMA), MALABO (CMA), LIBREVILLE, MONROVIA, NOUAKCHOTT, NIGERIA ports, SENEGAL DAKAR, FREETOWN, and more
  - Americas/Caribbean: PANAMA ports, HAITI ports, TRINIDAD & TOBAGO ports, PUERTO RICO, and more
  - Middle East: KHOR FAKKAN
- 2 new ports added: ALGERIA/ALGIERS (USD 6,200), ALGERIA/ALGIERS(40") (USD 9,800)
- Total port count: **636** (up from 634)

### 10. Port selection UX — selected port shown clearly
- Previously: after selecting a port, the user had to re-read the port list highlight or the small freight badge to know which port was chosen, which was easy to miss
- Now: clicking any port in the list immediately:
  1. Collapses the search panel (search box, region/country dropdowns, port list)
  2. Shows a prominent green confirmation box: **"✅ Port Selected — COUNTRY — PORT NAME | Freight: USD X/FCL | freshness"**
  3. Displays a **"Change Port"** button to re-open the search panel if needed
- `clearPortSelection()` function added — resets to search panel when "Change Port" is clicked

## v2026-05-16d — Joel's Amendments Batch 4

### 11. Customer Profiles — up to 100 saved customer templates

#### Overview
A full customer profile manager stored in the browser's localStorage. Profiles survive page refreshes and browser restarts. Each profile holds the customer's full usual requirements (SKUs, FCLs, mode, ports, additive, margin) so you can instantly load a pre-built quotation cart for any customer.

#### Key design: oil price is NEVER stored
When a profile is saved, the oil price is deliberately **not** saved. When a profile is loaded, every item is recalculated using the current CP10 price (`getOilPrice(oilName, S.cp10)`). This ensures quotes always reflect today's market — no stale prices.

#### How to use
1. Build a quote cart as normal (add items with their SKUs, FCLs, modes, ports)
2. Click **"💾 Save Profile"** (in the Quotation Report header) — fill in customer name, company, phone, email, notes → **Create Profile**
3. Next time: click **"👥 Customers"** → find the customer → **"⚡ Load Quote"**
4. Items load into the cart at current oil prices. Customer/Attn field is auto-filled.
5. If the cart already has items: prompted to Replace or Append

#### Update Prices button
A **"🔄 Update Prices"** button appears in the quote cart header whenever the cart has items. Clicking it recalculates every item's price to the current CP10 in one tap — useful when the oil price changes while the cart is open, or when loading from a profile and the price has shifted.

#### Profile management
- **Edit**: Update name, company, phone, email, notes (not items — reload + re-save to update items)
- **Delete**: Remove profile (confirmation required)
- **Search**: Filter profiles by name or company in real time
- Slot count displayed: `X/100 profiles used`

#### Storage
Profiles saved to `localStorage` under key `ablecalc_customers`. Each profile stores: `id, name, company, phone, email, notes, items[], createdAt, updatedAt`. Items include: `oilName, sku, fcl, add, additiveType, mgn, mode, port, priceMode` — but **not** `oilPrice` (always recalculated on load).
