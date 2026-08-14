# Dashboard — La scala

**Epic:** Dashboard
**Status:** v1 — Built
**Related:** [prd.md](../../prd.md) · [changelog.md](../../changelog.md) · [brand-system/brand-system.md](../../brand-system/brand-system.md)

---

## Goal

La prima e unica schermata di Netto. L'utente inserisce una RAL e ottiene il netto mensile, accompagnato dalla scala: la sequenza ordinata delle trattenute, ognuna chiamata con il nome che ha in busta paga, ognuna spiegabile.

L'epica copre l'intero prodotto v1: input, assunzioni, calcolo, risultato, ledger, spiegazioni. Non ci sono altre schermate, non c'è navigazione, non c'è account.

## Who it's for

Chi ha in mano un contratto con scritto `RAL 30.000 €` e non sa cosa arriva sul conto. In seconda battuta l'HR o il founder che deve spiegare quel numero senza sbagliarlo.

## How it works

Una pagina sola, quattro blocchi in verticale:

1. **Input** — RAL, mensilità, aliquota INPS.
2. **Assunzioni** — chip read-only che dichiarano tutto ciò che il calcolo dà per scontato.
3. **Risultato** — netto mensile come numero grande, netto annuo e totale trattenute sotto.
4. **La scala** — il ledger delle voci, ognuna espandibile con la spiegazione del calcolo.

Prima del calcolo esistono solo i blocchi 1 e 2. Risultato e scala compaiono dopo il click su *Calcola il netto* e si aggiornano a ogni ricalcolo senza ricaricare la pagina.

## Requirements

As a user, I want to:

| # | Requirement | Priority |
|---|---|---|
| R1 | Inserire la mia RAL in un unico campo, con il pulsante di calcolo attivo solo quando il valore è valido | P0 |
| R2 | Scegliere le due assunzioni che mi riguardano davvero — mensilità (13/14) e aliquota INPS (9,19% / 9,49%) — prima di calcolare | P0 |
| R3 | Vedere dichiarate a schermo, prima di calcolare, tutte le assunzioni fisse del modello | P0 |
| R4 | Vedere il netto mensile come numero grande, con netto annuo e totale trattenute subito sotto | P0 |
| R5 | Leggere la scala: ogni voce in ordine, con il nome istituzionale, l'aliquota applicata e l'importo allineato a destra | P0 |
| R6 | Espandere ogni voce per capire come è stata calcolata, in italiano semplice, con i miei numeri e la fonte normativa | P0 |
| R7 | Cambiare la RAL e rifare il calcolo senza ricaricare la pagina né perdere le mie scelte | P0 |
| R8 | Capire cosa ho sbagliato quando l'input non è valido, in una frase che dice anche come uscirne | P0 |
| R9 | Usare tutto da telefono, con le stesse informazioni e nessuno scroll orizzontale | P0 |
| R10 | Vedere la scala disegnarsi dall'alto verso il basso quando premo *Calcola*, così capisco che è una sottrazione | P1 |
| R11 | Sapere, quando il mio reddito ci rientra, che il cuneo fiscale mi aggiunge una somma esentasse invece di ridurre l'IRPEF | P1 |

## User flow

```
Stato iniziale
  → campo RAL vuoto, pulsante disabilitato
  → chip assunzioni visibili
  → messaggio: "Nessun calcolo ancora. Inserisci la tua RAL per iniziare."

Inserimento
  → utente digita RAL
  → RAL valida (1–300.000)      → pulsante attivo
  → RAL fuori range o = 0       → pulsante disabilitato + messaggio d'errore
  → utente può cambiare mensilità e aliquota INPS in qualsiasi momento

Calcolo
  → click su "Calcola il netto"
  → la scala si disegna dall'alto, il netto mensile conta fino al valore
  → risultato annunciato via aria-live

Lettura
  → click su una voce della scala → si espande con formula, numeri, fonte
  → più voci possono restare aperte insieme

Ricalcolo
  → utente cambia RAL, mensilità o aliquota
  → il risultato precedente resta visibile finché non si preme di nuovo Calcola
```

## Screen content

### Blocco input

| Elemento | Comportamento |
|---|---|
| Campo RAL | Numerico, simbolo € nel campo, formattazione italiana. Label: `Retribuzione annua lorda (RAL)` |
| Mensilità | Selettore 13 / 14. Default 13 |
| Aliquota INPS | Selettore 9,19% / 9,49%. Etichette: `Azienda ≤ 15 dipendenti` / `Azienda > 15 dipendenti (include FIS)`. Default 9,19% |
| Pulsante | `Calcola il netto`. Disabilitato con RAL vuota o non valida |

### Chip assunzioni (read-only)

`Impiegato/a a tempo indeterminato` · `Anno fiscale 2026` · `Milano · Lombardia` · `Anno intero (365 giorni)` · `Nessun familiare a carico` · `Nessuna agevolazione fiscale`

Stanno sopra il pulsante, non in un footer. Sono dichiarazioni, non controlli.

### Blocco risultato

```
NETTO MENSILE · SU 13 MENSILITÀ
1.801,96 €
23.425,51 € netti all'anno · 6.574,49 € di trattenute (21,9% della RAL)
```

L'etichetta segue la scelta dell'utente: `SU 14 MENSILITÀ` se ha selezionato 14.

### La scala

Voci in ordine, ognuna espandibile. Il segno è sempre esplicito.

| Voce | Segno | Note |
|---|---|---|
| Retribuzione lorda (RAL) | — | Riga di partenza |
| Contributi INPS `9,19%` | − | Mostra l'aliquota scelta |
| Imponibile fiscale | = | Riga di subtotale |
| IRPEF lorda | − | Mostra gli scaglioni attraversati |
| Detrazione lavoro dipendente `art. 13 TUIR` | + | |
| Ulteriore detrazione (cuneo fiscale) | + | Solo se imponibile 20.001–40.000 |
| IRPEF netta | = | Riga di subtotale |
| Addizionale regionale Lombardia | − | |
| Addizionale comunale Milano `0,80%` | − | Riga assente/a zero sotto i 23.000 di imponibile |
| Riduzione cuneo fiscale (somma esentasse) | + | Solo se imponibile ≤ 20.000, **dopo** i subtotali |
| **Netto annuo** | = | Riga conclusiva |

Le righe del cuneo sono mutuamente esclusive: o compare la detrazione, o compare la somma esentasse, mai entrambe.

### Pane "Come è stato calcolato?"

Ogni voce espansa mostra tre cose, in quest'ordine:

1. **La regola in italiano semplice** — *"L'IRPEF si calcola a scaglioni: il 23% sui primi 28.000 €, poi il 33% da 28.001 a 50.000 €, poi il 43% sul resto."*
2. **I tuoi numeri** — *"Il tuo imponibile di 27.243 € rientra tutto nel primo scaglione: 27.243 × 23% = 6.265,89 €."*
3. **La fonte** — *"Art. 11 TUIR, aliquote 2026 (L. 199/2025)."*

Seconda persona singolare, sentence case, nessun punto esclamativo.

## Copy

| Situazione | Testo |
|---|---|
| Stato vuoto | `Nessun calcolo ancora. Inserisci la tua RAL per iniziare.` |
| RAL < 1 | `Inserisci una RAL valida.` |
| RAL > 300.000 | `Netto supporta RAL fino a 300.000 €.` |
| Pulsante | `Calcola il netto` |
| Titolo risultato | `Netto mensile` |
| Trigger spiegazione | `Come è stato calcolato?` |
| Nota soglia 23.000 | `L'addizionale comunale di Milano prevede un'esenzione per redditi fino a 23.000 €. Superata questa soglia, si paga sull'intero imponibile.` |
| Nota addizionali | `Le addizionali si pagano l'anno successivo. Qui sono calcolate sullo stesso anno del reddito: è la semplificazione standard dei calcolatori di netto.` |

## Edge cases

| Caso | Comportamento atteso |
|---|---|
| Imponibile ≤ 8.500 | IRPEF netta = 0. Compare la riga somma esentasse. Netto ≈ RAL − INPS + somma esente |
| Imponibile ≤ 20.000 | Cuneo come somma esentasse, dopo i subtotali, spiegato come "va direttamente in tasca" |
| Imponibile 20.001–32.000 | Ulteriore detrazione fissa a 1.000 €, accanto alla detrazione art. 13 |
| Imponibile 32.001–40.000 | Ulteriore detrazione in phase-out, la spiegazione lo dice |
| Imponibile > 40.000 | Riga cuneo assente, non a zero |
| Imponibile ≤ 23.000 | Addizionale comunale a zero, con la nota sulla soglia |
| Imponibile > 50.000 | Detrazione lavoro dipendente a zero, terzo scaglione IRPEF attivo |
| RAL > 52.190 | INPS con l'1% aggiuntivo sopra soglia, visibile nella spiegazione della voce |
| Detrazioni > IRPEF lorda | IRPEF netta = 0, mai negativa. L'eccedenza si perde e la spiegazione lo dice |
| RAL con decimali | Accettata, arrotondata al centesimo in output |

## Design notes

Il brand system è contesto di progetto, non va ricopiato. I punti che questa schermata deve rispettare:

- **Un numero grande, tutti gli altri allineati.** Il netto mensile è l'unico display; ogni altra cifra sta in una colonna monospaced allineata a destra.
- **Blu tiene, rosso trattiene.** Il colore è semantico: importi trattenuti in rosso, importi che restano in blu, il netto in giallo. Il giallo compare una volta sola per schermata.
- **Tutte le cifre monospaced e tabular.** Formattazione italiana sempre: `1.801,96 €`, simbolo dopo la cifra.
- **Il segno meno è esplicito** su ogni trattenuta: il colore non è mai l'unico segnale.
- **Un solo momento di movimento:** la scala si disegna dall'alto al basso al click su Calcola, il numero grande conta fino al valore. Nient'altro si anima. `prefers-reduced-motion` mostra lo stato finale subito.
- **La scala è una tabella vera**, con caption, così uno screen reader legge voce e importo come coppia. Gli importi hanno `aria-live="polite"`.

## Technical considerations

- Le regole fiscali stanno in un file separato e leggibile (`fiscalRules.js` o equivalente), senza logica di interfaccia, con la fonte normativa citata in commento su ogni aliquota. È il file che un revisore apre per primo.
- Ogni funzione di calcolo restituisce sia il risultato sia i passaggi intermedi, così il pane "Come è stato calcolato?" si popola dagli stessi dati del ledger e non può divergere.
- Nessun backend, nessun account, nessuna persistenza: il calcolo è tutto client-side.
- Le percentuali di riferimento e le soglie sono nel PRD §7, con i due esempi verificati (RAL 30.000 e RAL 50.000) da usare come test di correttezza al centesimo.

## Out of scope

Trattamento integrativo, detrazioni per familiari a carico, detrazione regionale Lombardia €150, selettore regione/comune, fringe benefit, costo azienda, confronto tra RAL, URL condivisibile, dark mode, tassazione differenziata della 13ª/14ª. Motivazioni in [prd.md §10](../../prd.md).
