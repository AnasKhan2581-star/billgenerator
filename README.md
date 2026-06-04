# Anas Traders — Bill Generator

A single-file (`index.html`) GST tax-invoice generator for **Anas Traders** (textiles —
suits, sarees, fabric). It makes individual bills by hand, and has two hidden bulk tools:
an **Excel/CSV batch importer** and a **bank-statement → bills** generator. Everything runs
client-side in the browser — no server, no install. Just open `index.html`.

---

## 1. Business basics

| Thing | Value |
|---|---|
| Firm | Anas Traders, 4760/2 Ground Floor, Jaipuriya Building, Jogiwara, Nai Sarak, New Delhi 110006 |
| GSTIN | 07AGGPK5556M1ZX |
| GST rate | **5%** on all items |
| Products | **Unstitched Suit** (HSN 5810, unit pc), **Saree** (HSN 5407, pc), **Fabric** (HSN 5408, mtr) |

### GST / rate math (important)
- Rates are entered **GST-inclusive** (the round price the customer pays, e.g. ₹4,800).
- The bill back-calculates the **taxable (exclusive) rate** = `inclusiveRate ÷ 1.05`.
- Tax split: either **CGST 2.5% + SGST 2.5%** (default, local sales) or **IGST 5%**
  (toggle "Apply IGST instead of CGST + SGST" / per-bill IGST checkbox).
- **Grand Total (incl. GST) = sum of (qty × inclusive rate)** — always the round amount.
- "Amount in words" is generated automatically.

### Invoice numbering
- The next invoice number is stored in a browser cookie (`lastInvoice`) and auto-fills.
- Printing a single bill bumps it by 1. The statement tool has an optional
  "Update last invoice no. after building" checkbox.

---

## 2. Making a bill by hand (main UI)

1. Invoice No auto-fills; set the **Date**.
2. **Customer**: defaults to `Cash`. Tick **W** to expand the full **Bill To** box
   (Name, Address, State, State Code, GSTIN/UDIN).
3. **+ Add Item** → choose product, qty, and GST-inclusive rate. Add as many as needed.
4. Optionally tick **Apply IGST instead of CGST + SGST**.
5. **Generate Bill** → preview appears (HSN cells are editable if you need to tweak).
6. **Print / Save as PDF** opens the print view and prints; **Share Bill** shares.

### Copies dropdown (single-bill only)
Top-right corner of the bill card (where the "Duplicate" tag used to be) is a small dropdown:

| Option | Pages printed | Tags |
|---|---|---|
| **Original + Duplicate** *(default)* | 2 | Original, Duplicate |
| Original + Duplicate + Triplicate | 3 | Original, Duplicate, Triplicate |
| Only Duplicate | 1 | Duplicate |

> This dropdown affects **hand-made single bills only**. The bulk tools always print **one
> "Duplicate"-tagged page per bill**.

### Compact layout for long bills
When a bill has **5 or more line items**, rows and fonts automatically shrink so up to
~8–10 items still fit on **one page** (the "Proprietor" signature no longer spills onto a
second page). Bills with **4 or fewer items keep the normal roomy layout**.

---

## 3. Hidden tools (triple-click triggers)

Two features are hidden behind **triple-clicking** a header label:

| Triple-click on | Opens |
|---|---|
| **"Tax Invoice"** (under the address) | Excel/CSV **batch import** |
| **"Anas Traders"** (the big title) | Bank-**statement → bills** grid |

---

## 4. Bank statement → bills (triple-click "Anas Traders")

Turns a **Kotak bank statement PDF** into a table of ready-to-print bills, one per money
received, then feeds the normal print system.

### How credits are read
- The parser reads every transaction and identifies **money received (Deposit / Cr.)** by
  the **running balance going up** — this reliably ignores all withdrawals/payments out.
- Each credit becomes a bill whose **Grand Total = the credit amount**.

### Rules applied while reading credits
1. **₹50 floor** — any credit **under ₹50** (e.g. a ₹1 "test" payment) is **skipped
   entirely**. No bill is made for it, and it is never merged into another payment.
2. **Part-payment merge** — when the **same sender** pays **multiple times in a row on the
   same date** (e.g. SHARIYA AYAZ pays 2000 + 2000 + 1000), those are treated as **one
   sale** and merged into a **single bill** (₹5,000). The sender name is read from the
   transaction description (UPI/IMPS). Only consecutive, same-day, same-name, each-≥₹50
   payments merge.

### The editable grid
Top controls:
- **Base rates** for Suit / Saree / Fabric — pre-filled with the actual rates used across
  past bills; editable, and you can add more (comma-separated). Fabric is kept to 95/100/105/110.
- **Suit value %** (default 30), **Start Invoice No** (from cookie), **Customer** (default Cash).
- **Reconciliation line**: Σ of all bill totals vs Σ of statement credits — turns **green**
  when every bill matches and the totals tie out.

Each row (one bill):

| Column | Meaning |
|---|---|
| **+** | Insert an extra bill above this one (prompts for amount, auto-fills, renumbers) |
| **Inv No** | Auto-numbered from Start Invoice No |
| **Date** | From the statement (editable) |
| **Payer (statement)** | Sender name from the statement; a yellow **×N** badge shows how many part-payments were merged |
| **Customer** | Defaults to Cash; expand **▾ full details** to set Name/Address/State/State Code/GSTIN for that one bill |
| **Items** | Product · qty · incl-rate; add/remove items inline |
| **IGST** | Per-bill CGST+SGST vs IGST toggle |
| **Target** | The credit amount the bill should equal (editable) |
| **Grand Total** | Live-computed |
| **Match** | ✓ when Grand Total = Target, else shows the difference |
| **🗑** | Delete the bill (renumbers the rest) |

### Auto-fill logic
For each credit, items are generated to hit the **exact** amount:
- Aim for roughly **30% Suit value, 70% Saree + Fabric** value.
- Use the base rates (or nearby round inclusive values); **Fabric** closes the remainder
  exactly and only varies across **95/100/105/110**.
- Kept to **3 line items** (sometimes **4** for big bills ≥ ₹15,000) — quantities carry the
  value, like real bills. Everything stays editable; the Match badge flags any gap.

### Other behaviour
- **Autosave** — the grid, base rates and customer overrides are saved to the browser
  (`localStorage`), so a refresh doesn't lose work.
- **Build Bills** — converts the grid into the standard batch and opens the batch modal
  (below) for printing. Optional checkbox updates the `lastInvoice` cookie afterward.

---

## 5. Excel / CSV batch import (triple-click "Tax Invoice")

Upload a spreadsheet; each row becomes a bill in the batch modal.

**Recognised columns** (header names are flexible — alternatives shown):
- `BillNo` / `Bill No`
- `Date`
- `CustomerName` / `Customer` / `Name`, `Address`, `State`, `StateCode` / `State Code`,
  `GSTIN` / `GSTIN/UDIN`
- `UseIGST` / `IGST` → `yes` for IGST
- Up to 10 items per row: `Item{n}Type`, `Item{n}Qty`, `Item{n}RateInclGST` / `Item{n}Rate`
  (n = 1…10). Item type is matched to Suit/Saree/Fabric.

Rows with missing qty/rate are flagged and skipped (the rest still process).

---

## 6. Batch printing (shared by both bulk tools)

The batch modal lists every prepared bill with a status and a **🖨 Print / PDF** button per row:
- **Print All** — prints all ready bills in **descending invoice order** (highest invoice
  first, lowest last).
- **Per-row Print / PDF** — prints just that bill.
- Bulk bills are always tagged **"Duplicate"**, one page each.

---

## 7. Tech notes
- Single static file: `index.html`. Open directly or serve statically.
- Libraries (CDN): **PDF.js** (read statement PDFs), **SheetJS/xlsx** (read Excel/CSV).
- Printing opens a generated print window/tab — if your browser blocks pop-ups, allow them
  once for this page.
- State stored client-side: `lastInvoice` (cookie), `anasBillGridState` (localStorage).

---

## 8. Quick reference — defaults & thresholds

| Setting | Default / Rule |
|---|---|
| GST | 5% (CGST 2.5 + SGST 2.5, or IGST 5) |
| Inclusive → taxable | `÷ 1.05` |
| Single-bill copies | Original + Duplicate |
| Compact layout | when items ≥ 5 |
| Statement credit floor | ignore amounts < ₹50 |
| Part-payment merge | same date + same sender + consecutive + each ≥ ₹50 |
| Auto-fill value mix | ~30% Suit / ~70% Saree + Fabric |
| Auto-fill line items | 3 (4 if amount ≥ ₹15,000) |
| Fabric rate variants | 95 / 100 / 105 / 110 |
| Print All order | descending invoice number |
