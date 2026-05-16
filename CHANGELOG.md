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

## v2026-05-10 — Initial confirmed working build
- FOB / CIF / Both pricing modes — each item locks its mode when added to quote
- Save PDF opens a new tab → user taps "Save as PDF" button → Chrome print dialog
- Price/MT column toggle (YES/NO) affects PDF layout
- Each item stores its own oil price, additive, margin, port independently
- Image export (copyAsImage) via Canvas API for WhatsApp sharing
- 634 ports across all regions with freshness indicators
- Password protection via session storage
