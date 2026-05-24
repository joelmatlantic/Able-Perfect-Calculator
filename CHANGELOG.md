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

## v2026-05-16e — Joel's Amendments Batch 5

### 12. Oil Price Manager — solves non-palm oil price management

#### The problem
The 14 independent non-palm oils (SBR, SFR/Sunflower Refined, CAR/Corn Refined, CNR/CNC/Canola, Cotton Seed, Groundnut, Ghee AMF, CFAD) had hardcoded fixed prices in the code. These went stale as soon as markets moved, and there was no way to update them without a code push. Any customer profile containing these oils would use stale prices on load.

#### The solution: two-tier oil price system
- **CP10-linked oils** (26 oils): price = CP10 + fixed differential. Automatically recalculates when CP10 changes in Step 2. No action ever needed.
- **Independent non-palm oils** (14 oils): `getOilPrice()` now checks `localStorage['ablecalc_oil_prices']` first before falling back to the hardcoded default. Once updated in the manager, the custom price is used everywhere.

#### Oil Price Manager (🛢 Oil Prices button in Step 2)
Two sections:
1. **CP10-Linked Oils** (read-only): shows all 26 oils with current auto-calculated prices — just shows what you're getting for free from CP10
2. **Independent Non-Palm Oils** (editable): all 14 oils with editable price inputs, freshness indicators:
   - 🟢 Green: updated within 7 days
   - 🟡 Amber: updated 7–21 days ago
   - 🔴 Red: not updated in 21+ days
   - ⚪ Default: never been set (showing hardcoded fallback)

**Save All Prices** → saves all changed prices to localStorage with a timestamp. All ongoing calculations and profile loads immediately use the new prices.

**Reset to Defaults** → clears all custom prices, reverts to hardcoded fallbacks.

#### Stale price warning in Step 2
When you select a non-palm oil and its price is default or stale, a yellow warning banner appears below the oil selector, with a direct link to the Oil Price Manager.

#### Impact on Customer Profiles
Since profiles store `oilName` (not `oilPrice`), and `getOilPrice()` is called fresh on every profile load, updating oil prices in the manager automatically ensures all future profile loads use the latest prices — no profile editing required.

#### How to use day-to-day
1. Open the calculator and click **🛢 Oil Prices** in Step 2
2. Update all independent oil prices from your market data source
3. Click **Save All Prices**
4. All calculations and profile loads now use current prices
5. Repeat whenever prices change (use the freshness indicators as a reminder)

## v2026-05-23 — Data & Pricing Update (ABLE_UPDATED_EXHIBITION_CALCULATOR_VIETNAM_EXPO_2026-230526.xlsx)

### Packaging Costs — 87 SKUs updated
All 87 matched SKUs had their packaging cost/MT refreshed from the new Excel file. 104 SKUs total in the calculator (16 not present in the new Excel retained unchanged).

### Freight Rates — 35 ports updated + 1 new port
**Updated (Africa):** ANGOLA/LOBITO (3250→3650), ANGOLA/LUANDA (2250→2650), ANGOLA/NAMIBE (3250→3650), BENIN/COTONOU (1950→2350), CAMEROON/DOUALA (2750→3150), CAMEROON/KRIBI (2650→3050), COMOROS/MORONI (5750→6050), COMOROS/MUTSAMUDU (6250→6550), CONGO/MATADI (3050→3450), CONGO/POINTE NOIRE (2250→2550), COTE D'IVOIRE/ABIDJAN (1950→2350), COTE D'IVOIRE/SAN PEDRO (4150→4450), EQUATORIAL GUINEA/BATA CMA (3200→3500), EQUATORIAL GUINEA/MALABO CMA (3200→3500), GABON/LIBREVILLE (2650→4050), GAMBIA/BANJUL (3550→3850), GHANA/TEMA (2175→2550), GUINEA/BISSAU (4850→5150), GUINEA/CONAKRY (6800→7100), LIBERIA/MONROVIA (3600→3900), MAURITANIA/NOUAKCHOTT (3150→3350), NIGERIA/APAPA (2550→2950), NIGERIA/HARCOURT (4350→4650), NIGERIA/LAGOS TINCAN (2550→2950), NIGERIA/LEKKI (2550→2950), NIGERIA/ONNE SEA PORT (2550→2950), SENEGAL/DAKAR (3350→3650), SIERRA LEONE/FREETOWN (4050→4350), TOGO/LOME (1950→2350)

**Updated (Middle East):** ISRAEL/ASHDOD (4050→4250), ISRAEL/ASHDOD IND (4450→4550), JORDAN/AQABA (3750→3800), JORDAN/AQABA IND (6950→4100), SAUDI ARABIA/JEDDAH (3750→3500), SAUDI ARABIA/JEDDAH IND (4150→3800)

**New port added:** UAE / JEBEL ALI @ USD 4,850/FCL — Total: **637 ports**

### Oil Differentials (CP10-based) — 8 updated
| Oil | Old Diff | New Diff |
|-----|----------|----------|
| CPO | +70 | +85 |
| P. Stearin | -40 | -35 |
| P. Kernel Oil | +1165 | +1155 |
| P. Kernel Olein | +925 | +915 |
| Shortening 32-35deg | +30 | +25 |
| CBSFAT 33-35 | +1445 | +1455 |
| PFAD | -115 | -110 |
| PKFAD | +635 | +675 |

### Independent Oil Default Prices — 14 updated
| Oil | Old Default | New Default |
|-----|-------------|-------------|
| SBR | 1,560 | 1,510 |
| SBR w/o FTA | 1,380 | 1,370 |
| SBC | 1,550 | 1,500 |
| CAR | 1,660 | 1,620 |
| CAR w/o FTA | 1,460 | 1,500 |
| SFR | 1,880 | 1,730 |
| SFR w/o FTA | 1,760 | 1,650 |
| MZR | 1,640 | 1,670 |
| CNR | 2,450 | 2,080 |
| CNC | 2,400 | 2,010 |
| COTTON | 1,650 | 1,480 |
| GroundNut Oil | 2,900 | 2,050 |
| Ghee (AMF) | 7,500 | 6,700 |
| CFAD | 1,700 | 1,700 (unchanged) |

### CP10 Default Price
Updated from **USD 1,215/MT** → **USD 1,175/MT** (new selling price from Excel Sheet1)

## v2026-05-23b — Supplier Reference Rates Panel (Admin / User 0 Only)

### 13. Supplier Rates Modal — Apical, Wingagro, KLK

**Access:** "📊 Supplier Rates" button in the top header. Admin-only view (password-protected; only accessible after main login).

**Purpose:** Tracks the yellow reference input cells, per-packaging premium/discount, and packaging costs for all three suppliers, following the Excel formulas exactly (verified against `Apical_Price_List_13May2026-1.xlsx`).

#### Formula logic per supplier (from Excel audit)

**Apical** — `Price/MT = (BaseInput + Differential + PackagingCost) / Margin (0.955)`
- Malaysia CP10 (N2=1,175) → Jerry Cans (CP10/CP8), 20kg CBR 37/40
- Indonesia RBD PO (O2=N2−125=1,050) → All Shortenings, BIB Spread Fat, LOW GE Shortenings
- RBD Palm Kernel Oil (L2=2,180) → HPKO 35/37, HPKO 38/40, RPKO BIB
- RBD Palm Kernel Olein (K2=2,040) → HPKL 41/44, Apifil 7011 XC
- Coconut (R2=2,080) → RCNO, HCNO, RCNO Flexibag
- Stearin (S2=1,135) → HPS Flakes
- HPKS Stearin (Q2 was empty in Excel — defaulted to 2,350) → HPKS 33.5/35.5
- RBD Palm Kernel Stearin (M2=2,350) — labelled but NOT referenced in any formula ⚠

**Wingagro** — `Price/MT = (BaseInput + Differential + PackagingCost) / Margin (0.955)`
- Indonesia CP10 (M2=L2−20=1,155) → Jerry Cans CP10/CP8
- RBD Palm Kernel Stearin P2 (P2=2,360) → HPKS 33.5/35.5 (uses P2, NOT K2)
- RBD Palm Kernel Oil (J2=2,180) → HPKO 38/40
- I2 (PBKO Olein), K2 (PBKS), N2 (ID RBD PO), Q2 (Coconut), R2 (Stearin) — labelled but NOT used ⚠

**KLK** — `Price/MT = (CP10 + Differential + PackagingCost) / Margin (0.975)`
- CP10 (J3=1,175) drives ALL 80 products via `J_row = $J$3 + K_row`
- No issues found — all formulas correct

#### Features
- **Editable yellow inputs**: all oil prices, margin, Oillio margin — updates live calculations
- **Derived values shown**: Indonesia RBD PO (=CP10−125), Indonesia CP10 (=CP10−20)
- **Per-product editable table**: product name, base input label, premium/discount, packaging cost, auto-calculated price
- **Dirty highlighting**: changed inputs/cells highlighted in amber until saved
- **Save & Log**: saves all data to localStorage (`ablecalc_supplier_rates`), records timestamped diff log in `ablecalc_supplier_logs`
- **Log panel**: shows last 100 save events with timestamps, tabs affected, field-by-field changes
- Notes on known Excel formula issues (Q2 empty, unused inputs) shown inline
