# Netto — Brand System (condensed)

**Net Salary Calculator** · Converts RAL → netto mensile with every payslip line visible.

> **Parliamo netto.** Il prodotto mostra ogni riga tra il lordo del contratto e il netto in banca.

---

## Principi

1. **Show the subtraction** — mai un numero solo; sempre le voci che lo producono, in ordine, con l'aritmetica visibile.
2. **Use the State's own words** — IRPEF, addizionale regionale/comunale, contributi INPS, detrazioni. Mai "tasse" generico.
3. **Declare the simplifications** — le assunzioni (13 mensilità, Milano, 9,19%, no agevolazioni) sono on-screen, non in footnote.
4. **One number big, all others aligned** — il netto mensile è l'headline; tutto il resto in colonna monospaced allineata a destra.

---

## Colori

| Token | Hex | Ruolo |
|---|---|---|
| `--nt-blu` | `#1F3FD1` | Primary. Brand, risultati, importi rimanenti, azioni primarie |
| `--nt-blu-scuro` | `#142785` | Ombra bottone, stato pressed |
| `--nt-rosso` | `#E4402A` | Solo trattenute (INPS, IRPEF, addizionali) e errori |
| `--nt-giallo` | `#FFC845` | Razionato: barra netto, focus ring, max 1 highlight per schermo |
| `--nt-inchiostro` | `#14161B` | Tutto il body text, bordi input, codici sezione |
| `--nt-carta` | `#FAFAF6` | Sfondo pagina. Le card su `#FFFFFF` per stacco |
| `--nt-nebbia` | `#C9CEDB` | Righe, divider, linee tratteggiate, stati disabled |

**Semantica:** blu = ciò che tieni, rosso = ciò che viene trattenuto. Se un colore appare, fa un'affermazione sul denaro.
**Contrasto:** Inchiostro su Carta 15.4:1 · Carta su Blu 8.1:1 · Inchiostro su Giallo 11.9:1.
Rosso mai per body copy sotto 16 px; gli importi delle trattenute sempre ≥16 px con segno `−` esplicito.

---

## Tipografia

Tre font, tre ruoli. Ogni numero è sempre monospaced e tabular.

| Ruolo | Font | Setting |
|---|---|---|
| Display | **Archivo** Expanded 800 | `wdth 122`, tracking −0.045em, line-height 0.82 |
| Headings | **Archivo** 700 | `wdth 112`, tracking −0.025em |
| Body & UI | **Instrument Sans** 400/600 | 17 px, line-height 1.55 |
| Figures | **JetBrains Mono** 700 | `tabular-nums`, tracking −0.02em |
| Labels & codes | **JetBrains Mono** 400 | 11 px, tracking 0.08em, uppercase |

### Numeri

- Formato italiano sempre: `22.468,37 €` (simbolo dopo, spazio non-breaking).
- Due decimali nel ledger, zero nell'headline.
- `font-variant-numeric: tabular-nums` su ogni importo.
- Trattenute con `−` esplicito. Percentuali in mono: `9,19%`.

### Scala tipografica (1.25)

`--nt-t-xs: 11px` · `--nt-t-s: 13px` · `--nt-t-m: 17px` · `--nt-t-l: 21px` · `--nt-t-xl: 34px` · `--nt-t-hero: clamp(40px, 9vw, 56px)`

### Font import

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Archivo:wdth,wght@100..125,400..800&family=Instrument+Sans:wght@400..700&family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
```

---

## Tokens CSS

```css
:root {
  --nt-blu:        #1F3FD1;
  --nt-blu-scuro:  #142785;
  --nt-rosso:      #E4402A;
  --nt-giallo:     #FFC845;
  --nt-inchiostro: #14161B;
  --nt-carta:      #FAFAF6;
  --nt-nebbia:     #C9CEDB;

  --nt-display: "Archivo", system-ui, sans-serif;
  --nt-body:    "Instrument Sans", system-ui, sans-serif;
  --nt-mono:    "JetBrains Mono", ui-monospace, monospace;

  --nt-t-xs: 11px;  --nt-t-s: 13px;  --nt-t-m: 17px;
  --nt-t-l: 21px;   --nt-t-xl: 34px; --nt-t-hero: clamp(40px, 9vw, 56px);

  --nt-1: 4px; --nt-2: 8px; --nt-3: 12px; --nt-4: 16px;
  --nt-6: 24px; --nt-8: 32px; --nt-12: 48px;

  --nt-r-key: 10px;  --nt-r-card: 14px;  --nt-r-mark: 23%;
  --nt-ease: cubic-bezier(.2,.75,.3,1);
  --nt-dur: 500ms;   --nt-stagger: 90ms;
}
.amount { font-family: var(--nt-mono); font-variant-numeric: tabular-nums; }
```

---

## Il mark

Quattro barre discendenti su un tasto arrotondato. Legge come stack di sottrazione e come righello di busta paga. L'ultima barra è gialla: è ciò che tieni.

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

Wordmark in Archivo Expanded 800. Clear space = 1 altezza barra. Min size: 20 px (mark), 96 px (lockup).

---

## La scala (signature visiva)

La stessa idea è logo, loading state e grafico risultato. Ogni step mostra il rimanente in blu e ciò che è stato sottratto in rosso. L'ultimo step diventa giallo (netto). Le barre sono proporzionali alla RAL, mai riscalate per riga. Su mobile gli importi vanno sotto la barra.

---

## Bottoni & interazione

- **Bottone primario:** sfondo `--nt-blu`, testo `--nt-carta`, border-radius `--nt-r-key` (10px). Al press scende 2 px sulla propria ombra (`--nt-blu-scuro`) — effetto tasto calcolatrice.
- **Focus ring:** 3 px `--nt-giallo`, 3 px offset, visibile su ogni controllo.
- **Assunzioni come chip** sopra il bottone (13 mensilità, Milano, ecc.), editabili.

### Motion

- Su **Calcola**: la scala si disegna top→bottom, 90 ms stagger per step, 500 ms ciascuno, `cubic-bezier(.2,.75,.3,1)`.
- Il numero headline conta su in 400 ms. Nient'altro anima.
- `prefers-reduced-motion`: stato finale istantaneo.

---

## Voice & copy

Italiano, sentence case, seconda persona singolare, zero punti esclamativi. Il tono è un collega competente che spiega la busta paga: preciso, mai apologetico, mai da venditore.

| Sì | No |
|---|---|
| Calcola il netto | Scopri ora il tuo stipendio! |
| Inserisci una RAL tra 10.000 € e 300.000 € per vedere il calcolo. | Ops! Qualcosa è andato storto. |
| Questo calcolo assume 13 mensilità e aliquote di Milano. | I risultati sono puramente indicativi. |
| Nessun calcolo ancora. Inserisci la tua RAL per iniziare. | Nessun dato disponibile. |

**Termini fiscali:** nomi legali sempre (IRPEF, non "imposta sul reddito"). Ciò che l'utente controlla usa nomi piani: `mensilità`, non `periodicità di pagamento`.
**Errori:** descrivi cosa è successo e la via d'uscita, in una frase. Niente scuse, niente colpe.
**Coerenza:** il bottone dice "Calcola il netto", il risultato dice "Netto mensile" — la stessa parola attraversa il flusso.

---

## Accessibilità

- Il colore non veicola mai significato da solo: le trattenute hanno segno `−` e label.
- Il ledger è un `<table>` con caption; screen reader legge label+importo in coppia.
- Importi con `aria-live="polite"`.
- Body text mai sotto 15 px; input font-size ≥16 px (previene zoom iOS).
