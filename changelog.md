# Changelog

Feature changelog by date, newest first. Source: [netto-salary-calculator-web](https://github.com/) (Lovable app repo).

---

## 2026-08-14

### Dashboard
- Dashboard v1 completa su rotta unica `/` (`src/routes/index.tsx`): input RAL, selettore mensilità 13/14, selettore aliquota INPS 9,19% / 9,49%, chip assunzioni read-only, pulsante *Calcola il netto*. Stato vuoto con il messaggio previsto. `602ceb6`
- Blocco risultato (`Risultato.tsx`): netto mensile come numero grande con `aria-live`, netto annuo, totale trattenute in euro e in percentuale sulla RAL. `d44aec2`
- La scala (`Scala.tsx`): ledger come tabella vera con `caption`, segno esplicito per riga, badge aliquota, importi monospaced allineati a destra. `d44aec2` `31b5dfd`
- Pane "Come è stato calcolato?": ogni riga espandibile mostra regola, numeri personali e fonte normativa, alimentata dai passaggi intermedi delle funzioni di calcolo. Più righe restano aperte insieme. `5096411`
- Movimento: la scala si disegna riga per riga (`--row-index`, 90 ms di stagger), il netto mensile conta fino al valore in 400 ms. `prefers-reduced-motion` porta subito allo stato finale, sia per la scala sia per il count-up. `1e4039c`
- Validazione input: parsing italiano (`30.000`, `30000,50`, `30 000`), errori `Inserisci una RAL valida.` sotto 1 € e `Netto supporta RAL fino a 300.000 €.` oltre il massimo, pulsante disabilitato quando l'input non è valido. `e9fc209`

### Motore fiscale
- `src/lib/fiscalRules.ts` — tutte le regole dell'anno d'imposta 2026 isolate dalla UI, con la fonte normativa in commento su ogni aliquota: INPS (9,19% / 9,49% + 1% IVS oltre 52.190), IRPEF a scaglioni 23/33/43, detrazione lavoro dipendente art. 13 TUIR con comma 1-bis e capienza, cuneo fiscale (somma esentasse ≤ 20.000 / ulteriore detrazione 20.001–40.000), addizionale regionale Lombardia a 4 scaglioni, addizionale comunale Milano 0,80% con esenzione a 23.000. Ogni funzione restituisce risultato e passaggi intermedi. `e9fc209` `5096411`
- `src/lib/format.ts` — formattazione italiana (`1.801,96 €`, simbolo dopo la cifra con spazio non separabile) e parsing dell'input. `e9fc209`

### Brand
- Token del brand system in `src/styles.css`: utility `figure-num`, `label-code`, `display-num`, `wordmark`, `key-press`; colori semantici `text-keep` / `text-withheld`. Font Archivo, Instrument Sans, JetBrains Mono via Google Fonts. `8069abf`
- Marchio Netto come componente (`NettoLogo.tsx`) e favicon derivata dall'SVG di brand. `8069abf` `d5e8abd` `fdd4a14` `83b2035`

### Infra
- Meta tag e Open Graph su root e su `/`, titolo `Netto — calcola il netto mensile dalla tua RAL (2026)`. `d3ace17` `602ceb6`
- README di progetto nel repo del codice. `b3d8473`

---

## 2026-08-13

### Infra
- Scaffold iniziale da template Lovable `tanstack_start_ts_current`: TanStack Start + React 19, Tailwind CSS 4, componenti shadcn/ui, TanStack Query, Vite 8. Nessun backend, nessun database. `6b4c0f2`
