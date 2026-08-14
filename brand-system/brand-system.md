# Netto — Brand System v1.0

**Net Salary Calculator** · prepared for the Jet HR product builder task

> **Parliamo netto.**
> A contract gives you one number. Your bank account gives you another. Netto shows you every line between the two — named the way the payslip names it.

---

## VOCE 010 — The story

Every employee in Italy meets the same wall. The offer letter says `RAL 30.000 €`. The payslip says `1.728,34 €`. Nobody explains the distance, so people either guess, ask a colleague, or paste the number into an ad-heavy calculator that returns a single figure with no reasoning attached.

### The name

**Netto** is the amount that arrives — and, in Italian, the way you say something plainly. *Parlare netto*: to speak without hedging. The product does both jobs at once, which is why the name is the promise. `Net Salary Calculator` stays as the descriptor in the lockup, so the category is never in doubt.

### The position

Netto isn't a tax tool for accountants. It's a **reading aid for the person being paid**, and for the HR or founder who has to explain a number they didn't design. It turns a contract figure into a payslip you can read before you sign one.

### Four principles

| # | Principle | What it means in practice |
|---|---|---|
| 01 | **Show the subtraction** | A result is never a lone number. Every output carries the line items that produced it, in order, with the arithmetic visible. |
| 02 | **Use the State's own words** | IRPEF, addizionale regionale, addizionale comunale, contributi INPS, detrazioni. Not "taxes" and "fees". The words match the payslip so the user can find them there. |
| 03 | **Declare the simplifications** | This is a model, not a payroll engine. Assumptions (13 mensilità, Milano, aliquota 9,19%, no agevolazioni) are on screen, not in a footnote. |
| 04 | **One number big, all others aligned** | The monthly net is the headline. Everything else is a right-aligned monospaced column you can subtract by eye. |

---

## VOCE 020 — The mark

Four bars, descending, on a rounded key. It reads as a subtraction stack, as a payslip's ruled lines, and as the keycap of an Olivetti desk calculator — the Italian lineage this product belongs to. **The last bar is yellow because the last bar is what you keep.**

```
┌─────────────────────┐
│                     │
│   ███████████████   │  60 units  — RAL
│                     │
│   ███████████       │  45 units  — dopo contributi
│                     │
│   ████████          │  33 units  — dopo IRPEF
│                     │
│   █████             │  23 units  — NETTO (giallo)
│                     │
└─────────────────────┘
   100 × 100 · radius 23
```

### Lockups

| Variant | Use |
|---|---|
| **Primary** — mark in Blu Somma + wordmark, descriptor beneath | Default, on Carta |
| **Inverse** — mark and wordmark in Carta | On Inchiostro or Blu Somma |
| **App icon** — mark alone, 1:1, blue field | Favicon, avatar, mobile icon |
| **Monochrome** — single-colour bars, yellow bar becomes 60% tint | Fax-grade contexts, embossing, single-colour print |

### Construction

Built on a 100-unit square, corner radius 23. Bars are 11 units tall on a 17,5-unit rhythm, widths **60 / 45 / 33 / 23** — a fixed ratio, never re-drawn by hand. Clear space on all sides equals one bar height. Minimum size 20 px (mark alone), 96 px wide (full lockup).

### Misuse

- ✕ Don't reorder the bars ascending — the mark is a deduction, not growth.
- ✕ Don't recolour the yellow bar. It is the only yellow in the system.
- ✕ Don't set the wordmark in anything but Archivo Expanded 800.
- ✕ Don't add a drop shadow, gradient or outline to the key.

---

## VOCE 030 — Colour

Two working colours and a neutral pair, borrowed from Giovanni Pintori's Olivetti posters: flat blocks of ink blue and vermilion on poster stock. In Netto the split is semantic, not decorative — **blue is what you keep, red is what is withheld**. If a colour appears, it is making a claim about money.

| Name | Hex | Role |
|---|---|---|
| **Blu Somma** | `#1F3FD1` | Primary. Brand, results, remaining amounts, primary action. |
| **Rosso Trattenuta** | `#E4402A` | Deductions only — INPS, IRPEF, addizionali. Also errors. |
| **Giallo Tasto** | `#FFC845` | Rationed. The net bar, focus rings, one highlight per screen. |
| **Inchiostro** | `#14161B` | All body text, input borders, section codes. |
| **Carta** | `#FAFAF6` | Page ground. Cards sit on pure `#FFFFFF` to lift off it. |
| **Nebbia** | `#C9CEDB` | Rules, dividers, dashed ledger lines, disabled states. |
| *Blu Scuro* | `#142785` | Support only: the shadow under a pressed key. |

**Contrast:** Inchiostro on Carta 15.4:1 · Carta on Blu Somma 8.1:1 · Inchiostro on Giallo 11.9:1.
Rosso is never used for body copy below 16 px — deduction amounts are set at 16 px minimum and always carry a minus sign, so colour is never the only signal.

---

## VOCE 040 — Type

Three faces, three jobs. A payslip is a monospaced document — figures have to stack in a column so the eye can do the subtraction. That constraint decides the system: **every number in Netto is monospaced and tabular**, always.

| Role | Face | Setting |
|---|---|---|
| Display | **Archivo** Expanded 800 | `wdth 122`, tracking −0.045em, line-height 0.82 |
| Headings | **Archivo** 700 | `wdth 112`, tracking −0.025em |
| Body & UI | **Instrument Sans** 400/600 | 17 px, line-height 1.55 |
| Figures | **JetBrains Mono** 700 | tabular-nums, tracking −0.02em |
| Labels & codes | **JetBrains Mono** 400 | 11 px, tracking 0.08em, uppercase |

All three are SIL Open Font License.

### Why these three

Archivo's expanded widths carry the squared, engineered feel of Italian mid-century technical lettering (Novarese's Eurostile) without the retro costume. Instrument Sans is narrow and quiet at 15–17 px, so long fiscal sentences stay readable. JetBrains Mono is the ledger face — and its name is a small nod to where this prototype is going.

### Number rules

- Italian formatting always: `22.468,37 €`. Symbol after the figure, non-breaking space.
- Two decimals in the ledger, zero decimals in headline figures.
- `font-variant-numeric: tabular-nums` on every amount.
- Deductions always carry an explicit `−`. Percentages in mono: `9,19%`.

---

## VOCE 050 — The signature: *la scala*

One idea used three times. The descending staircase is **the logo, the loading state, and the result chart** — so the thing you see on the icon is literally the thing the product does. Each step keeps the remaining amount in blue and shows what was just removed in red, left-aligned, so the blue edge walks down and to the left. The final step turns yellow: that's the net.

```
RAL                 ████████████████████████████████████  30.000,00
− INPS 9,19%        ████████████████████████████████▓▓▓▓  −2.757,00
− IRPEF netta       ███████████████████████████▓▓▓▓▓      −4.221,60
− Add. regionale    ██████████████████████████▓           −335,09
− Add. comunale     ██████████████████████████▓           −217,94
                    ══════════════════════════
= NETTO             ██████████████████████████            22.468,37 €
                    (giallo)
```

Bars are proportional to the RAL, never rescaled per row — the whole point is that the small ones look small. On mobile the amounts drop below each bar rather than shrinking the bar.

---

## VOCE 060 — The product

The whole interface is one input, one action, one result, one ledger. Assumptions are visible as chips above the button, because the fastest way to lose trust in a calculator is to hide what it assumed.

```
┌────────────────────────────────────────────┐
│  ▪ Netto                                   │
├────────────────────────────────────────────┤
│  Retribuzione annua lorda (RAL)            │
│  ┌──────────────────────────────────────┐  │
│  │ €  30.000                            │  │
│  └──────────────────────────────────────┘  │
│  (Impiegato · t. indet.) (13 mensilità)    │
│  (Milano · Lombardia) (Nessuna agevol.)    │
│  ┌──────────────────────────────────────┐  │
│  │          Calcola il netto            │  │
│  └──────────────────────────────────────┘  │
├────────────────────────────────────────────┤
│  NETTO MENSILE · SU 13 MENSILITÀ           │
│  1.728,34 €                                │   ← Blu Somma field
│  22.468,37 € netti all'anno ·              │
│  7.531,63 € di trattenute                  │
├────────────────────────────────────────────┤
│  Retribuzione lorda              30.000,00 │
│  Contributi INPS  9,19%          −2.757,00 │
│  Imponibile fiscale              27.243,00 │
│  IRPEF lorda  23%                −6.265,89 │
│  Detrazioni lav. dip.  art.13    +2.044,29 │
│  Addizionale regionale  1,23%      −335,09 │
│  Addizionale comunale   0,80%      −217,94 │
│  ══════════════════════════════════════════│
│  Netto annuo                   22.468,37 € │
└────────────────────────────────────────────┘
```

*Figures are an illustrative worked example for the visual system, not the calculation spec. They use the 9,19% INPS rate, the 23% scaglione and the art. 13 TUIR detrazione, and deliberately ignore the 2025 ulteriore detrazione for the 20.000–32.000 band — worth about €1.000/year and an explicit decision for the engine.*

---

## VOCE 070 — Voice & copy

Interface Italian, sentence case, second person singular, no exclamation marks. The voice is a competent colleague explaining your payslip at a desk: precise about the mechanism, never apologetic, never selling.

| Say | Not |
|---|---|
| Calcola il netto | Invia / Scopri ora il tuo stipendio! |
| Inserisci una RAL tra 10.000 € e 300.000 € per vedere il calcolo. | Ops! Qualcosa è andato storto. |
| Questo calcolo assume 13 mensilità e aliquote di Milano. Cambia le assunzioni se non è il tuo caso. | I risultati sono puramente indicativi e non vincolanti. |
| Nessun calcolo ancora. Inserisci la tua RAL per iniziare. | Nessun dato disponibile. |

- **Naming things** — fiscal terms keep their legal names. Everything the user controls gets a plain name: `mensilità`, not `payment periodicity`.
- **Errors** — state what happened and the way out, in one sentence. No apology, no blame, no vague *riprova più tardi*.
- **Consistency** — the button says **Calcola il netto**; the result heading says **Netto mensile**. The same word travels through the flow.

---

## VOCE 080 — Motion & access

### Motion

- One orchestrated moment only: on **Calcola**, the scala draws top to bottom, 90 ms stagger per step, 500 ms each, `cubic-bezier(.2,.75,.3,1)`.
- The headline figure counts up over 400 ms. Nothing else animates.
- Buttons press down 2 px onto their own shadow — a rubber calculator key, not a hover glow.
- `prefers-reduced-motion` renders the final state instantly.

### Access

- Focus ring: 3 px Giallo Tasto, 3 px offset, visible on every control.
- Colour never carries meaning alone — deductions have a minus sign and a label.
- The ledger is a real `<table>` with a caption, so a screen reader reads label and amount as a pair.
- Amounts get `aria-live="polite"`; the result is announced, not just repainted.
- Body text never below 15 px; input font-size 16 px+ to stop iOS zoom.

---

## VOCE 090 — Tokens

```css
/* Netto — design tokens v1.0 */
:root {
  /* colore — blue keeps, red withholds */
  --nt-blu:        #1F3FD1;   /* primary, remaining, actions   */
  --nt-blu-scuro:  #142785;   /* key shadow, pressed state     */
  --nt-rosso:      #E4402A;   /* trattenute, errors            */
  --nt-giallo:     #FFC845;   /* the net bar, focus ring       */
  --nt-inchiostro: #14161B;   /* text, borders                 */
  --nt-carta:      #FAFAF6;   /* page ground                   */
  --nt-nebbia:     #C9CEDB;   /* rules, dividers, disabled     */

  /* carattere */
  --nt-display: "Archivo", system-ui, sans-serif;      /* wdth 110–122, wght 700–800 */
  --nt-body:    "Instrument Sans", system-ui, sans-serif;
  --nt-mono:    "JetBrains Mono", ui-monospace, monospace;

  /* scala tipografica — 1.25 */
  --nt-t-xs: 11px;  --nt-t-s: 13px;  --nt-t-m: 17px;
  --nt-t-l: 21px;   --nt-t-xl: 34px; --nt-t-hero: clamp(40px, 9vw, 56px);

  /* spazio — 4px base */
  --nt-1: 4px; --nt-2: 8px; --nt-3: 12px; --nt-4: 16px;
  --nt-6: 24px; --nt-8: 32px; --nt-12: 48px;

  /* forma & moto */
  --nt-r-key: 10px;  --nt-r-card: 14px;  --nt-r-mark: 23%;
  --nt-ease: cubic-bezier(.2,.75,.3,1);
  --nt-dur: 500ms;   --nt-stagger: 90ms;
}

/* every amount, everywhere */
.amount { font-family: var(--nt-mono); font-variant-numeric: tabular-nums; }
```

### Web font import

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Archivo:wdth,wght@100..125,400..800&family=Instrument+Sans:wght@400..700&family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
```

### The mark, as SVG

```html
<svg viewBox="0 0 100 100" role="img" aria-label="Netto">
  <rect width="100" height="100" rx="23" fill="#1F3FD1"/>
  <g fill="#FAFAF6">
    <rect x="20" y="23"   width="60" height="11" rx="3.5"/>
    <rect x="20" y="40.5" width="45" height="11" rx="3.5"/>
    <rect x="20" y="58"   width="33" height="11" rx="3.5"/>
  </g>
  <rect x="20" y="75.5" width="23" height="11" rx="3.5" fill="#FFC845"/>
</svg>
```

---

**Netto** — brand system v1.0.
Type: Archivo, Instrument Sans, JetBrains Mono (SIL OFL). Colour lineage: Giovanni Pintori's Olivetti poster work, 1949–1967.
