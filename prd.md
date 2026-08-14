# Netto — PRD v1 & Calculation Spec

**Product:** Netto — Net Salary Calculator
**Tagline:** Parliamo netto.
**Author:** [your name]
**Date:** August 2026
**Tax year modelled:** 2026 (anno d'imposta 2026, L. 199/2025 — Legge di Bilancio 2026)

---

## 1. Problem

An Italian employment contract states a RAL (Retribuzione Annua Lorda). The person signing it — whether the employee or the HR/founder explaining the offer — cannot read what that number means in practice without mentally running a fiscal pipeline they were never taught. Existing calculators either produce a single number with no breakdown, or expose every possible toggle and bury the user in options they can't evaluate.

Netto turns a RAL into a readable ledger: a descending sequence of labelled deductions, each named in the State's own terminology, ending in the monthly net. It is a reading aid, not a tax tool.

## 2. User & JTBD

**Primary user:** A person evaluating a job offer or trying to understand their current payslip — typically 25–45, employed or about to be, not a tax professional.

**Secondary user:** An HR person or founder at a small/mid Italian company who needs to explain "here's what you'll actually take home" without getting the number wrong.

**Job to be done:** "When I receive a contract with a RAL, I want to see what lands in my bank account each month, and understand what was subtracted and why, so I can evaluate the offer without calling an accountant."

## 3. Non-goals

These are explicitly excluded from the prototype and acknowledged as simplifications:

- TFR (it's deferred compensation, not a payslip deduction the employee sees monthly)
- Detrazioni per familiari a carico (requires modelling dependents — separate feature)
- Trattamento integrativo / ex bonus Renzi (complex capienza check, affects only redditi ≤ 28.000 with specific conditions; noted as v2)
- Fringe benefit, buoni pasto, welfare, premi di risultato
- Part-time, apprendistato, contratto a termine specifics
- Regimi agevolati (impatriati, under-30, Mezzogiorno)
- CCNL-specific elements beyond mensilità count
- Multi-employer / multi-income scenarios
- Employer cost view (costo azienda)
- Real-time payslip simulation (monthly accruals, conguaglio)

## 4. Success criteria (not a launch)

1. **Arithmetic correctness:** The worked example at RAL 30.000 matches the spec to the cent. A reviewer can change the RAL and verify each intermediate step against the formulas in the config file.
2. **Transparency:** Every deduction line is visible, labelled with its institutional name, and the assumptions are declared on-screen before the user clicks.
3. **Simplification legibility:** The README explains what was included, what was excluded, and why — a reviewer understands the scoping decisions in under two minutes.
4. **Code auditability:** The fiscal rules live in one file (`fiscalRules.js` or equivalent), separate from UI logic, with comments citing the legal source for each rate.

## 5. Inputs (v1)

| Input | Type | Default | Notes |
|---|---|---|---|
| RAL | Number (€) | empty | The only required input. Range: 1–300.000 |
| Mensilità | Dropdown: 13 / 14 | 13 | Determines monthly net division only |
| Aliquota INPS | Toggle: 9,19% / 9,49% | 9,19% | Label: "Azienda ≤15 dip." / "Azienda >15 dip." |

## 6. Visible assumptions (chips, on-screen above the button)

- Impiegato/a a tempo indeterminato
- Anno fiscale 2026
- Residente a Milano, Lombardia
- Anno intero lavorato (365 giorni)
- Nessun familiare a carico
- Nessuna agevolazione fiscale

These are displayed as read-only chips in v1. The mensilità dropdown and INPS toggle are the only assumptions that become real inputs.

---

## 7. Calculation spec

This is the ordered pipeline. Each step shows: the formula, the rates, the legal source, and a confidence flag (✅ verified against 2026 sources, ⚠️ requires periodic check, ❓ uncertain / needs further verification).

### Step 1 — Contributi INPS a carico del lavoratore

**What it is:** Mandatory pension contributions withheld from gross pay.

**Formula:**
```
if RAL ≤ 52.190:
  contributi_inps = RAL × inps_rate
else:
  contributi_inps = 52.190 × inps_rate + (RAL − 52.190) × (inps_rate + 0.01)
```

**Rates:**
- Base rate: 9,19% (aziende ≤ 15 dip.) or 9,49% (aziende > 15 dip., includes 0,30% FIS)
- Additional 1% on retribuzione exceeding €52.190/year (contributo aggiuntivo IVS, art. 3-ter L. 438/1992)

**Source:** L. 335/1995; INPS circolare annuale aliquote contributive FPLD. Threshold €52.190 from INPS tabelle 2026.
**Confidence:** ✅ Rate confirmed for 2026. ⚠️ The €52.190 threshold is indexed annually — verify against INPS circolare for anno 2026.

**Notes for prototype:**
- The 1% surcharge only kicks in at RAL ≈ €57.000+ (since 52.190 is on retribuzione, not RAL, but for our simplified model RAL = retribuzione). Worth implementing because the input accepts up to 300k.
- The 9,19% vs 9,49% toggle label should explain: "Includi contributo FIS (aziende >15 dipendenti)".

---

### Step 2 — Imponibile fiscale (reddito complessivo)

**Formula:**
```
imponibile = RAL − contributi_inps
```

**Source:** Art. 10 TUIR — contributi previdenziali obbligatori are deducible from reddito complessivo.
**Confidence:** ✅

**Notes:** In our simplified single-income model, imponibile fiscale = reddito complessivo = reddito da lavoro dipendente. This equivalence breaks if the user has other income sources — out of scope.

---

### Step 3 — IRPEF lorda

**What it is:** National progressive income tax.

**Formula:**
```
irpef_lorda = 0
if imponibile > 0:
  irpef_lorda += min(imponibile, 28000) × 0.23
if imponibile > 28000:
  irpef_lorda += min(imponibile − 28000, 22000) × 0.33
if imponibile > 50000:
  irpef_lorda += (imponibile − 50000) × 0.43
```

**Rates (2026):**

| Scaglione | Aliquota | Su reddito da... a... |
|---|---|---|
| 1° | 23% | 0 — 28.000 |
| 2° | 33% | 28.001 — 50.000 |
| 3° | 43% | oltre 50.000 |

**Source:** Art. 11, co. 1 TUIR, as modified by art. 1, co. 3, L. 199/2025 (Legge di Bilancio 2026). Second bracket reduced from 35% to 33%.
**Confidence:** ✅ Structural from 2026.

---

### Step 4 — Detrazione per lavoro dipendente (art. 13 TUIR)

**What it is:** A tax credit that reduces IRPEF for employment income. Applied automatically by employer.

**Formula (full year, 365 days):**
```
if imponibile ≤ 15000:
  detrazione_base = 1955
elif imponibile ≤ 28000:
  detrazione_base = 1910 + 1190 × (28000 − imponibile) / 13000
elif imponibile ≤ 50000:
  detrazione_base = 1910 × (50000 − imponibile) / 22000
else:
  detrazione_base = 0

// Detrazione aggiuntiva comma 1-bis
if 25000 ≤ imponibile ≤ 35000:
  detrazione_base += 65

// Cap: detrazione cannot exceed IRPEF lorda
detrazione_lavdip = min(detrazione_base, irpef_lorda)
```

**Source:** Art. 13, co. 1 and co. 1-bis TUIR, as modified by D.Lgs. 216/2023, confirmed by L. 207/2024 and L. 199/2025.
**Confidence:** ✅ Confirmed unchanged for 2026.

**Notes:**
- The formula intentionally produces values > 1.955 in the 15.001–28.000 range (up to ~3.100 at reddito 15.001). This is correct, not a bug.
- The +65€ (comma 1-bis) applies to redditi 25.000–35.000. It was introduced as part of the fiscal reform and confirmed for 2026.
- Minimum guarantee (690€ for t. indeterminato, 1.380€ for t. determinato) exists but only matters for very short employment periods. Since we assume full year, we can ignore the minimum in v1.

---

### Step 5 — Ulteriore detrazione / Cuneo fiscale

**What it is:** A structural tax benefit introduced by L. 207/2024 and made permanent by L. 199/2025. Two separate mechanisms depending on income level.

**Formula:**
```
// Mechanism A: Somma esentasse (non-taxable bonus, added to net)
// Only for reddito ≤ 20.000
if imponibile ≤ 8500:
  somma_esente = imponibile × 0.071
elif imponibile ≤ 15000:
  somma_esente = imponibile × 0.053
elif imponibile ≤ 20000:
  somma_esente = imponibile × 0.048
else:
  somma_esente = 0

// Mechanism B: Ulteriore detrazione (reduces IRPEF)
// Only for reddito 20.001–40.000
if 20000 < imponibile ≤ 32000:
  ulteriore_detrazione = 1000
elif 32000 < imponibile ≤ 40000:
  ulteriore_detrazione = 1000 × (40000 − imponibile) / 8000
else:
  ulteriore_detrazione = 0

// Cap: ulteriore_detrazione cannot make IRPEF go below zero
ulteriore_detrazione = min(ulteriore_detrazione, irpef_lorda − detrazione_lavdip)
// (ensure non-negative)
ulteriore_detrazione = max(ulteriore_detrazione, 0)
```

**Source:** Art. 1, co. 4 (somma esentasse) and co. 6 (ulteriore detrazione), L. 207/2024. Confirmed structural by L. 199/2025.
**Confidence:** ✅ Confirmed for 2026.

**Critical implementation note:** Mechanism A and Mechanism B are mutually exclusive (different income ranges). Mechanism A is NOT a tax credit — it's a non-taxable sum added directly to net pay. Mechanism B IS a tax credit that reduces IRPEF. The UI should reflect this difference:
- For Mechanism A: show as "+ Riduzione cuneo fiscale" after the net subtotal
- For Mechanism B: show as "+ Ulteriore detrazione (cuneo fiscale)" alongside the art. 13 detrazione

**For the "Come è stato calcolato?" explanation pane:**
- Mechanism A: "Il cuneo fiscale aggiunge al tuo netto una somma esentasse pari al X% del reddito. Non è una detrazione: non riduce l'IRPEF, va direttamente in tasca."
- Mechanism B: "L'ulteriore detrazione riduce l'IRPEF di €X. È una misura strutturale che si aggiunge alla detrazione per lavoro dipendente."

---

### Step 6 — IRPEF netta

**Formula:**
```
irpef_netta = irpef_lorda − detrazione_lavdip − ulteriore_detrazione
irpef_netta = max(irpef_netta, 0)  // IRPEF cannot be negative
```

**Notes:** If total detrazioni exceed IRPEF lorda, the excess is lost — no refundable credit in the standard case (trattamento integrativo would handle this, but that's v2).

---

### Step 7 — Addizionale regionale (Lombardia)

**What it is:** Regional income surtax, progressive by brackets.

**Formula:**
```
add_regionale = 0
if imponibile > 0:
  add_regionale += min(imponibile, 15000) × 0.0123
if imponibile > 15000:
  add_regionale += min(imponibile − 15000, 13000) × 0.0158
if imponibile > 28000:
  add_regionale += min(imponibile − 28000, 22000) × 0.0172
if imponibile > 50000:
  add_regionale += (imponibile − 50000) × 0.0173
```

**Rates (Lombardia, 2026):**

| Scaglione | Aliquota |
|---|---|
| 0 — 15.000 | 1,23% |
| 15.001 — 28.000 | 1,58% |
| 28.001 — 50.000 | 1,72% |
| oltre 50.000 | 1,73% |

**Source:** Art. 72, L.R. 10/2003 (Lombardia); scaglioni confirmed on regione.lombardia.it for periodo d'imposta 2026.
**Confidence:** ✅ Scaglioni verified on official regional site.

**Notes:**
- Lombardia also offers a detrazione regionale of €150 for redditi 28.001–50.000 (L.R. 2/2025). ⚠️ I have verified the existence of this detrazione but not its exact mechanics. Recommendation: EXCLUDE from v1, note in README as a known omission (~€150 overcharge for affected band). Include in v2.
- There are additional regional detrazioni for families with 2+ children (€100/child). Out of scope per "nessun familiare a carico."
- The addizionale is technically paid the year after (acconto/saldo cycle). For simplicity, we model it as same-year — this is the standard approach for net salary calculators and matches what the employee experiences in busta paga.

---

### Step 8 — Addizionale comunale (Milano)

**What it is:** Municipal income surtax.

**Formula:**
```
if imponibile > 23000:
  add_comunale = imponibile × 0.008
else:
  add_comunale = 0  // esenzione, NOT franchigia
```

**Rate:** 0,80% flat (aliquota unica)
**Esenzione:** redditi imponibili ≤ €23.000 — the entire addizionale is zeroed. This is NOT a franchigia: once you exceed €23.000, you pay on the full imponibile, not just the excess.

**Source:** Deliberazione Consiglio Comunale Milano n. 36 del 21/10/2013; confirmed on comune.milano.it for 2026. No new delibera published for 2026 → 2025 rate carries over.
**Confidence:** ✅ Rate confirmed. ⚠️ Esenzione threshold confirmed at €23.000 from official municipal page.

**Same-year simplification:** Same as addizionale regionale — modelled in the same tax year.

---

### Step 9 — Netto annuo

**Formula:**
```
netto_annuo = RAL
             − contributi_inps
             − irpef_netta
             − add_regionale
             − add_comunale
             + somma_esente  // only if imponibile ≤ 20.000 (cuneo Mechanism A)
```

---

### Step 10 — Netto mensile

**Formula:**
```
netto_mensile = netto_annuo / mensilità   // 13 or 14
```

**Note on 13ª/14ª mensilità taxation:** In reality, the tredicesima/quattordicesima is taxed differently — detrazioni per lavoro dipendente are not applied to extra monthly payments. This means the actual monthly net for the 12 "ordinary" months is slightly higher, and the 13ª/14ª is slightly lower, than the simple division suggests. This is a known simplification. The annual net total is correct; only the monthly distribution is approximate.

---

## 8. Worked example — RAL 30.000, INPS 9,19%, 13 mensilità

| # | Voce | Formula | Amount |
|---|---|---|---|
| 1 | Retribuzione lorda (RAL) | input | 30.000,00 |
| 2 | − Contributi INPS (9,19%) | 30.000 × 9,19% | −2.757,00 |
| 3 | = Imponibile fiscale | 30.000 − 2.757 | 27.243,00 |
| 4 | − IRPEF lorda | 27.243 × 23% | −6.265,89 |
| 5 | + Detrazione lavoro dipendente | 1.910 + 1.190 × (28.000 − 27.243) / 13.000 + 65 | +2.044,28 |
| 6 | + Ulteriore detrazione (cuneo) | 1.000 (fascia 20.001–32.000) | +1.000,00 |
| 7 | = IRPEF netta | 6.265,89 − 2.044,28 − 1.000 | −3.221,61 |
| 8 | − Addizionale regionale Lombardia | (15.000 × 1,23%) + (12.243 × 1,58%) | −377,94 |
| 9 | − Addizionale comunale Milano | 27.243 × 0,80% | −217,94 |
| 10 | **= Netto annuo** | 30.000 − 2.757 − 3.221,61 − 377,94 − 217,94 | **23.425,51** |
| 11 | **÷ 13 = Netto mensile** | 23.425,51 / 13 | **1.801,96** |

**Totale trattenute:** 6.574,49 (21,9% della RAL)

---

## 9. Second worked example — RAL 50.000, INPS 9,19%, 13 mensilità

This tests multi-bracket IRPEF, detrazione phase-out, and cuneo phase-out.

| # | Voce | Formula | Amount |
|---|---|---|---|
| 1 | RAL | | 50.000,00 |
| 2 | − INPS (9,19%) | 50.000 × 9,19% | −4.595,00 |
| 3 | = Imponibile | | 45.405,00 |
| 4 | − IRPEF lorda | (28.000 × 23%) + (17.405 × 33%) | −12.183,65 |
| 5 | + Detrazione lav. dip. | 1.910 × (50.000 − 45.405) / 22.000 | +398,89 |
| 6 | + Ulteriore detr. (cuneo) | 1.000 × (40.000 − 45.405) / 8.000 → negative → 0 | +0,00 |
| 7 | = IRPEF netta | 12.183,65 − 398,89 | −11.784,76 |
| 8 | − Add. regionale | (15k×1,23%)+(13k×1,58%)+(17.405×1,72%) | −689,17 |
| 9 | − Add. comunale | 45.405 × 0,80% | −363,24 |
| 10 | **= Netto annuo** | | **32.567,83** |
| 11 | **÷ 13** | | **2.505,22** |

**Notes on this example:**
- No +65€ (reddito 45.405 > 35.000)
- No cuneo (reddito 45.405 > 40.000)
- Detrazione is small (398,89) and approaching zero (hits zero at 50.000)
- Two IRPEF brackets used (23% + 33%)

---

## 10. Feature scoping

### v1 — Must ship

| Feature | Rationale |
|---|---|
| RAL input with validation (1–300.000) | Core |
| Mensilità dropdown (13/14, default 13) | Per user decision |
| INPS rate toggle (9,19%/9,49%) | Per user decision |
| Assumption chips (read-only) | Brand principle: "Declare the simplifications" |
| Calcola button | Core |
| Result block: monthly net (big), annual net, total withheld | Brand principle: "One number big" |
| Ledger (la scala): ordered deduction lines with amounts | Brand principle: "Show the subtraction" |
| "Come è stato calcolato?" expandable pane per each voce | Per user decision — explains each line in plain Italian |
| Responsive layout (works on mobile) | Interview quality signal |
| Fiscal rules in a separate auditable file | Success criterion #4 |

### v2 — Deliberate future (mentioned in README)

| Feature | Why not v1 |
|---|---|
| Trattamento integrativo (ex bonus Renzi) | Complex capienza check; only affects redditi ≤ 28.000 with specific conditions. For no-dependents, no-other-deductions case, it rarely triggers in the 15k–28k range. |
| Detrazione regionale Lombardia €150 | Verified existence but not exact mechanics; ~€150 impact for redditi 28k–50k |
| Regione / Comune selectors | Requires a database of ~8.000 comuni × aliquote. Huge scope increase. |
| Detrazioni familiari a carico | Requires modelling dependents, ages, Assegno Unico interaction |
| Employer cost view (costo azienda) | Different audience, different calculation |
| RAL comparison mode (side by side) | Useful but not core |
| Shareable result URL (RAL in query param) | Nice UX, easy to add |
| 14ª mensilità differential taxation | Correct monthly distribution (12 months + bonus month taxed differently) |
| Dark mode | Design system supports it (Inchiostro/Carta swap) |
| Fringe benefit (campo opzionale) | Si somma all'imponibile. Un campo, zero complessità fiscale aggiuntiva — ma aggiunge un input che va spiegato. |

### Out of scope entirely

TFR, buoni pasto (sotto soglia sono esenti e non entrano nel calcolo), welfare, part-time, apprendistato, regimi agevolati, CCNL-specific elements beyond mensilità, multi-income, real-time monthly simulation.

---

## 11. Edge cases and validation

### Input validation

| Condition | Behaviour |
|---|---|
| Empty input | Button disabled; no error shown |
| Non-numeric input | Prevent input (input type="number" or mask) |
| RAL < 1 | Show: "Inserisci una RAL valida" |
| RAL > 300.000 | Show: "Netto supporta RAL fino a 300.000 €" — this is a product decision, not a fiscal one. Above ~200k there are additional detrazione caps (€440 forfettario) we don't model. |
| RAL with decimals | Accept, round to nearest cent in output |
| RAL = 0 | Button disabled |

### Fiscal edge cases the code must handle

| Case | What happens | RAL range |
|---|---|---|
| Imponibile ≤ 8.500 (no-tax area) | Detrazione ≥ IRPEF lorda → IRPEF netta = 0. Cuneo somma esente applies. Net ≈ RAL − INPS + somma esente. | RAL ≲ 9.400 |
| Imponibile crosses 15.000 | Detrazione formula switches. Test at RAL ~16.600. | ~16.500–16.700 |
| Imponibile crosses 23.000 | Addizionale comunale Milano kicks in (esenzione → full tax, not gradual). Creates a small "cliff." | RAL ~25.400 |
| Imponibile crosses 28.000 | IRPEF second bracket starts. Detrazione formula switches. Cuneo switches from 1.000 fixed to unchanged (still in 20k–32k band for reddito 28k). | RAL ~30.900 |
| Imponibile crosses 32.000 | Cuneo starts decreasing. | RAL ~35.300 |
| Imponibile crosses 35.000 | +65€ detrazione aggiuntiva stops. | RAL ~38.600 |
| Imponibile crosses 40.000 | Cuneo = 0. | RAL ~44.100 |
| Imponibile crosses 50.000 | Detrazione = 0. IRPEF third bracket starts. | RAL ~55.100 |
| RAL crosses 52.190 | INPS additional 1% kicks in. | ~52.200+ |
| Imponibile crosses 20.000 | Cuneo switches from Mechanism A (somma esente) to Mechanism B (detrazione). There's a design discontinuity here. | RAL ~22.000 |

### The 23.000 cliff (Milano)

This is a real edge case worth highlighting in the UI. At imponibile 23.000, addizionale comunale = 0. At imponibile 23.001, addizionale comunale = 23.001 × 0,8% = 184 €. A €1 increase in reddito costs €184 in addizionale. The prototype should handle this correctly. The "Come è stato calcolato?" pane should explain: "L'addizionale comunale di Milano prevede un'esenzione per redditi fino a 23.000 €. Superata questa soglia, si paga sull'intero imponibile."

---

## 12. Build recommendation

### Stack

**Plain HTML + CSS + vanilla JS.** No framework. Reasons:

1. The brief explicitly says "the point is not how well I use Lovable or similar tools, but that I've built something whose logic I understand and control." A framework adds abstraction between you and the logic.
2. The fiscal calculation is ~80 lines of arithmetic. There is no state management problem. No components to reuse. No routing.
3. A reviewer can open one HTML file and one JS file and see everything. No build step, no node_modules, no package.json to distract.
4. Deploys trivially to GitHub Pages.

### File structure

```
netto/
├── index.html           # Single page, all markup
├── css/
│   └── style.css        # Design system implementation
├── js/
│   ├── fiscalRules.js   # THE file — all rates, formulas, sources
│   └── app.js           # DOM interaction, formatting, rendering
├── fonts/               # Archivo Expanded, Instrument Sans, JetBrains Mono (or Google Fonts links)
├── README.md            # The artefact the reviewer reads first
└── SOURCES.md           # Optional: full bibliography of legal sources
```

### `fiscalRules.js` — the auditable file

This is the file the interviewer will open. Structure it as:

```javascript
/**
 * Netto — Fiscal Rules Configuration
 * Tax year: 2026 (anno d'imposta 2026)
 * 
 * Every rate and formula cites its legal source.
 * This file contains NO DOM logic — it exports pure functions.
 */

// === INPS ===
// Source: L. 335/1995; INPS FPLD tables 2026
const INPS_RATE_STANDARD = 0.0919;  // aziende ≤ 15 dipendenti
const INPS_RATE_FIS = 0.0949;       // aziende > 15 dip. (+ 0,30% FIS)
const INPS_SURCHARGE_THRESHOLD = 52190; // Additional 1% above this
const INPS_SURCHARGE_RATE = 0.01;

// === IRPEF 2026 ===
// Source: Art. 11 TUIR, modified by L. 199/2025 art. 1 co. 3
const IRPEF_BRACKETS = [
  { limit: 28000, rate: 0.23 },
  { limit: 50000, rate: 0.33 },
  { limit: Infinity, rate: 0.43 },
];

// ... etc.

// Each function: calculateINPS(ral, rate), calculateIRPEF(imponibile), etc.
// Each returns an object with both the result and the intermediate steps,
// so the UI can populate the "Come è stato calcolato?" pane.
```

### Formatting rules

- All monetary values displayed with Italian formatting: `1.801,96 €`
- Use `Intl.NumberFormat('it-IT', ...)` for consistency
- All figures in JetBrains Mono, `font-variant-numeric: tabular-nums`
- Deductions right-aligned with `−` prefix (minus sign, not hyphen)
- Credits with `+` prefix

---

## 13. README structure

```markdown
# Netto — Net Salary Calculator

Parliamo netto.

## What this is

Netto turns an Italian RAL (gross annual salary) into a readable monthly net,
showing every deduction by name, in order. It models tax year 2026 for a
standard case: impiegato a tempo indeterminato, resident in Milano.

## Try it

[Live site URL] or open `index.html` locally.

## How it works

Enter a RAL → see your monthly net and the full breakdown.

The calculation follows this pipeline:
RAL → INPS → Imponibile → IRPEF → Detrazioni → Cuneo fiscale → Addizionali → Netto

Each step is documented in `js/fiscalRules.js` with legal sources.

## What's included

- IRPEF 2026 (3 scaglioni: 23% / 33% / 43%)
- Contributi INPS (9,19% or 9,49%, user-selectable)
- Detrazione lavoro dipendente (art. 13 TUIR, all 4 bands + comma 1-bis)
- Cuneo fiscale strutturale (somma esentasse ≤ 20k / ulteriore detrazione 20k–40k)
- Addizionale regionale Lombardia (4 scaglioni progressivi)
- Addizionale comunale Milano (0,80% flat, esenzione ≤ 23.000)

## What's deliberately excluded (and why)

| Excluded | Why |
|---|---|
| Trattamento integrativo (ex bonus Renzi) | Complex capienza check; in v1's no-dependents case, rarely triggers for redditi 15k–28k |
| Detrazioni familiari | Requires modelling dependents — different feature |
| TFR | Deferred compensation, not a payslip deduction |
| Regione/Comune selector | ~8.000 comuni, each with own rates — would need a database |
| Detrazione regionale Lombardia €150 | Verified existence, not exact mechanics — ~€150 impact |
| Fringe benefit, welfare, buoni pasto | Employer-specific, not derivable from RAL |
| Part-time, apprendistato, agevolazioni | Each is a separate calculation model |

## Assumptions

Declared on-screen. The user sees them before clicking "Calcola."

## Sources

All rates cite their legal source in `js/fiscalRules.js`. Key references:
- IRPEF: Art. 11 TUIR, L. 199/2025
- Detrazioni: Art. 13 TUIR, D.Lgs. 216/2023
- Cuneo: Art. 1 co. 4 and co. 6, L. 207/2024
- INPS: L. 335/1995, INPS circolari 2026
- Add. regionale: Art. 72 L.R. 10/2003 (Lombardia)
- Add. comunale: Del. CC Milano n. 36/2013

## Tech

Vanilla HTML/CSS/JS. No build step. No dependencies.
`fiscalRules.js` contains all rates and formulas — start there to audit the logic.
```

---

## 14. The "Come è stato calcolato?" pane

Implementation recommendation: each voce in the ledger gets an expandable row (click/tap to expand). The expanded content shows:

1. **The formula in plain Italian** — e.g., "L'IRPEF si calcola a scaglioni: il 23% sui primi 28.000 €, poi il 33% sulla parte da 28.001 a 50.000 €, poi il 43% sul resto."
2. **Your specific numbers** — e.g., "Il tuo imponibile di 27.243 € rientra tutto nel primo scaglione: 27.243 × 23% = 6.265,89 €."
3. **The source** — e.g., "Art. 11 TUIR, aliquote 2026 (L. 199/2025)."

Keep it in second person singular, sentence case, no exclamation marks — consistent with the brand voice.

For the cuneo fiscale, explain the mechanism clearly:
- ≤ 20.000: "Il cuneo fiscale ti riconosce una somma esentasse pari al X% del tuo reddito. Non è una detrazione: questi soldi vanno direttamente nel tuo netto, senza essere tassati."
- 20.001–32.000: "L'ulteriore detrazione riduce la tua IRPEF di 1.000 €. È una misura strutturale introdotta dalla Legge di Bilancio 2025 e confermata per il 2026."
- 32.001–40.000: same as above but note the phase-out.

---

## 15. Open items / risks

Questa sezione elenca le cose che sappiamo di non sapere con certezza, o dove abbiamo fatto una scelta che potrebbe essere contestata. Sono nel documento proprio perché in un colloquio è meglio dire "questo lo so e ho deciso così" che farsi trovare impreparati.

### 15.1 INPS threshold (€52.190)

L'aliquota INPS del 9,19% si applica sulla retribuzione fino a un tetto. Sopra quel tetto scatta un 1% aggiuntivo. Il problema è che il tetto viene aggiornato ogni anno dall'INPS con una circolare, legato alla rivalutazione. Il valore €52.190 è citato in diverse fonti 2026, ma non è stata trovata la circolare INPS originale che lo conferma per l'anno d'imposta 2026 esatto.

**Impatto:** nel range tipico (RAL sotto 50k) non cambia nulla. Siccome accettiamo RAL fino a 300k, il codice usa questo tetto. Se il numero vero fosse 53.000 o 51.800, l'errore sarebbe di qualche euro per RAL altissime.
**Mitigazione:** prima di consegnare, cercare la circolare INPS esatta. Flaggare nel codice con un commento.

### 15.2 Detrazione regionale €150 (Lombardia)

La Regione Lombardia prevede una detrazione di €150 dall'addizionale regionale per chi ha reddito tra 28.001 e 50.000 (introdotta con L.R. 2/2025). L'esistenza è verificata, ma non la meccanica esatta — se si applica sull'intera addizionale, se ha condizioni aggiuntive, se è rapportata a qualcosa. Quindi è esclusa.

**Impatto:** per chi ha imponibile in quella fascia, il nostro risultato sovrastima l'addizionale regionale di ~€150. È visibile.
**Mitigazione:** dichiarato nel README come omissione nota, così il revisore sa che ne siamo consapevoli.

### 15.3 Addizionali modellate nello stesso anno

Nella realtà le addizionali regionale e comunale funzionano con un ciclo acconto/saldo: quelle del 2025 si pagano in busta paga nel 2026, spalmate su 11 rate (gennaio–novembre). Quindi in un cedolino reale di gennaio 2026 si trovano trattenute per le addizionali del 2025, non del 2026. Noi le calcoliamo come se si pagassero nello stesso anno del reddito.

**Impatto:** il netto annuo totale è corretto, ma se qualcuno confronta con un singolo cedolino mensile reale i numeri non tornano, perché il cedolino mostra le addizionali dell'anno prima.
**Mitigazione:** è la semplificazione standard che fanno tutti i calcolatori di stipendio netto. Va spiegata nel pane "Come è stato calcolato?".

### 15.4 Trattamento integrativo omesso

A RAL molto basse (sotto ~16.500, imponibile sotto 15.000), il trattamento integrativo (ex bonus Renzi) vale fino a 1.200 €/anno — sono 100 €/mese in più nel netto. Noi non lo calcoliamo. Per il caso default del prototipo (nessun familiare a carico, nessuna altra detrazione) e redditi sopra 15k, il TI quasi mai scatta: serve che la somma delle detrazioni qualificate superi l'IRPEF lorda, e con la sola detrazione lavoro dipendente non succede.

**Impatto:** se qualcuno inserisce RAL 12.000, il nostro netto è sottostimato di circa 100 €/mese. È tanto.
**Mitigazione:** documentato nel README e segnato come prima feature da aggiungere in v2.

### 15.5 Cuneo Mechanism A — percentuali (7,1% / 5,3% / 4,8%)

Le tre percentuali per la somma esentasse sotto i 20.000 € vengono dalla L. 207/2024. Tutte le fonti 2026 le ripetono, ma nessuna dice esplicitamente "confermate invariate per il 2026" — tutte dicono "rese strutturali", che dovrebbe significare che non cambiano, ma "strutturale" in legislazione fiscale italiana a volte dura meno del previsto.

**Impatto:** basso. Se le percentuali fossero diverse, l'errore sarebbe di qualche decina di euro per RAL sotto 22k.
**Mitigazione:** ⚠️ verificare contro una fonte primaria (Agenzia delle Entrate o circolare INPS) prima della consegna.

### 15.6 Arrotondamenti

Ogni passaggio del calcolo arrotonda a 2 decimali. In un cedolino reale il datore di lavoro arrotonda in momenti diversi, e su base mensile non annuale (calcola IRPEF ogni mese sulla retribuzione mensile, non dividendo l'annuale per 12/13/14).

**Impatto:** differenze di qualche centesimo rispetto a un cedolino vero.
**Mitigazione:** non è risolvibile senza simulare il cedolino mese per mese, che è fuori scope. Si nota nel README e basta.

### 15.7 Fringe benefit esclusi da v1

I fringe benefit (auto aziendale, telefono, alloggio) sono reddito imponibile a tutti gli effetti: il valore convenzionale si somma all'imponibile e ci si pagano IRPEF e addizionali. Non sono un caso di nicchia — l'auto aziendale è comune. Sono esclusi da v1 per contenere lo scope, ma sono in v2 come campo opzionale ("Fringe benefit annui €") che si somma all'imponibile prima del calcolo IRPEF.

**Impatto:** chi ha fringe benefit significativi (auto aziendale: tipicamente 2.000–5.000 €/anno di valore convenzionale) vedrà un netto sovrastimato.
**Mitigazione:** segnalato nelle assumption chips e nel README. La v2 risolve con un singolo campo aggiuntivo.
