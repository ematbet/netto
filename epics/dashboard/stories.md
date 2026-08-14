# Dashboard — Stories

Epic: [epic.md](epic.md)

Ogni story è un prompt atomico per Lovable. Il brand system e il PRD sono allegati come contesto di progetto: non vanno ricopiati nel prompt.

**Sequenza:** S1 → S2 → S3. S1 è già un prodotto funzionante; S2 e S3 aggiungono profondità e rifinitura.

---

## S1 — Prima versione: input, calcolo, scala

> Crea la prima versione di Netto. Vorrei partire dal dashboard, seguendo i requirements:
>
> As a user, I want to:
>
> 1. Inserire la mia RAL (retribuzione annua lorda) in un unico campo, e vedere il pulsante "Calcola il netto" attivarsi solo quando ho inserito un valore valido tra 1 e 300.000 €.
> 2. Scegliere le due sole assunzioni che mi riguardano prima di calcolare: le mensilità (13 o 14, default 13) e l'aliquota INPS (9,19% per aziende fino a 15 dipendenti, 9,49% per aziende oltre i 15 dipendenti, default 9,19%).
> 3. Vedere dichiarate a schermo, sopra il pulsante e prima di calcolare, tutte le assunzioni fisse del calcolo: impiegato/a a tempo indeterminato, anno fiscale 2026, Milano · Lombardia, anno intero lavorato, nessun familiare a carico, nessuna agevolazione fiscale. Sono dichiarazioni in sola lettura, non controlli.
> 4. Vedere il mio netto mensile come numero grande e isolato, con sotto il netto annuo e il totale delle trattenute in euro e in percentuale sulla RAL. L'etichetta deve dire su quante mensilità è diviso.
> 5. Leggere la scala: la sequenza ordinata di tutte le voci che portano dalla RAL al netto annuo, ognuna con il suo nome istituzionale (contributi INPS, imponibile fiscale, IRPEF lorda, detrazione lavoro dipendente, ulteriore detrazione o somma esentasse del cuneo fiscale, IRPEF netta, addizionale regionale Lombardia, addizionale comunale Milano, netto annuo), l'aliquota applicata dove esiste, e l'importo con segno esplicito − o + allineato a destra.
> 6. Cambiare la RAL o le assunzioni e ripremere Calcola per aggiornare il risultato, senza ricaricare la pagina e senza perdere quello che avevo scelto.
> 7. Capire cosa devo correggere quando l'input non è valido: "Inserisci una RAL valida." sotto 1 €, "Netto supporta RAL fino a 300.000 €." sopra il massimo.
> 8. Vedere, prima di aver calcolato qualsiasi cosa, il messaggio "Nessun calcolo ancora. Inserisci la tua RAL per iniziare." al posto del risultato.
> 9. Usare tutto dal telefono, con le stesse informazioni e senza scroll orizzontale.
>
> Note:
> - Le formule fiscali, le aliquote, le soglie e i due esempi verificati (RAL 30.000 → netto mensile 1.801,96 € su 13 mensilità; RAL 50.000 → 2.505,22 €) sono nel PRD, sezioni 7, 8 e 9. Il risultato deve corrispondere al centesimo.
> - Tieni tutte le regole fiscali in un file separato dalla logica di interfaccia, con la fonte normativa citata in commento accanto a ogni aliquota. Ogni funzione di calcolo deve restituire anche i passaggi intermedi, non solo il totale.
> - Tutte le cifre in formato italiano: `1.801,96 €`, simbolo dopo la cifra. Le trattenute portano sempre il segno meno esplicito.
> - Nessun backend, nessun account: il calcolo è tutto client-side.

**Acceptance criteria**

- RAL 30.000 · 13 mensilità · 9,19% → netto mensile `1.801,96 €`, netto annuo `23.425,51 €`, trattenute `6.574,49 €`.
- RAL 50.000 · 13 mensilità · 9,19% → netto mensile `2.505,22 €`, netto annuo `32.567,83 €`.
- Con campo vuoto o RAL = 0 il pulsante è disabilitato e non compare nessun errore.
- Le regole fiscali sono in un file isolato, senza riferimenti al DOM.

---

## S2 — "Come è stato calcolato?"

> Aggiungi alla scala di Netto la spiegazione per ogni voce.
>
> As a user, I want to:
>
> 1. Espandere ogni riga della scala con un click per capire da dove viene quel numero.
> 2. Leggere, in ogni riga espansa e in quest'ordine: la regola in italiano semplice, poi il calcolo con i miei numeri, poi la fonte normativa.
> 3. Tenere aperte più righe insieme, per confrontarle.
> 4. Capire il caso della soglia dei 23.000 €: se il mio imponibile la supera, la spiegazione dell'addizionale comunale mi dice che Milano esenta i redditi fino a 23.000 € e che oltre quella soglia si paga sull'intero imponibile, non solo sull'eccedenza.
> 5. Capire quale meccanismo di cuneo fiscale mi si applica: se il mio imponibile è fino a 20.000 € il cuneo mi aggiunge una somma esentasse che va direttamente nel netto senza essere tassata; se è tra 20.001 e 40.000 € è invece una detrazione che riduce l'IRPEF.
> 6. Sapere che le addizionali regionale e comunale, nella realtà, si pagano l'anno dopo, e che qui sono calcolate sullo stesso anno del reddito.
>
> Note:
> - Il testo delle spiegazioni usa i passaggi intermedi già restituiti dalle funzioni di calcolo, così non può divergere dai numeri mostrati nella scala.
> - Voce: seconda persona singolare, sentence case, nessun punto esclamativo. Esempio per l'IRPEF: "L'IRPEF si calcola a scaglioni: il 23% sui primi 28.000 €, poi il 33% da 28.001 a 50.000 €, poi il 43% sul resto. Il tuo imponibile di 27.243 € rientra tutto nel primo scaglione: 27.243 × 23% = 6.265,89 €. Art. 11 TUIR, aliquote 2026 (L. 199/2025)."

**Acceptance criteria**

- Ogni voce della scala è espandibile e mostra regola, numeri personali e fonte.
- Con imponibile ≤ 20.000 compare la riga somma esentasse dopo i subtotali e mai la riga ulteriore detrazione; sopra i 40.000 nessuna delle due compare.
- La nota sulla soglia dei 23.000 € appare solo quando l'imponibile la supera.

---

## S3 — Movimento e accessibilità

> Rifinisci il dashboard di Netto.
>
> As a user, I want to:
>
> 1. Vedere la scala disegnarsi dall'alto verso il basso quando premo "Calcola il netto", una riga dopo l'altra, così capisco che sto guardando una sottrazione.
> 2. Vedere il netto mensile contare fino al valore finale invece di comparire di colpo.
> 3. Non vedere nessun'altra animazione nella pagina.
> 4. Ottenere il risultato finale immediatamente, senza animazioni, se ho attivato la riduzione del movimento nel sistema operativo.
> 5. Navigare tutto da tastiera, con il focus sempre visibile su ogni controllo.
> 6. Sentirmi leggere dallo screen reader ogni voce della scala come coppia etichetta + importo, e sentire annunciato il risultato quando cambia.
>
> Note:
> - La scala deve essere una tabella vera con caption, non righe di div.
> - Il colore non deve mai essere l'unico segnale: ogni trattenuta ha già il segno meno e l'etichetta.

**Acceptance criteria**

- Con `prefers-reduced-motion` attivo la pagina passa direttamente allo stato finale.
- Il risultato è annunciato via `aria-live` a ogni ricalcolo.
- Ogni controllo ha un focus ring visibile.
