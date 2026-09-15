# Dashboard bollette luce, gas e acqua

Dashboard statica (HTML + Chart.js) che legge i dati da `data.json`. Pensata per GitHub Pages:
aggiornare i numeri ogni mese significa sostituire un solo file, senza toccare `index.html`.

Online: <https://mark1395.github.io/Dashboard-Bollette/>

## Setup iniziale (una tantum, ~5 minuti)

1. Vai su [github.com/new](https://github.com/new) e crea un repository. Pubblico va bene:
   **nessun dato personale è presente** in questi file (niente nome, indirizzo, POD/PDR, codice
   fiscale o numero cliente — solo periodo, fornitore, consumi e costi).
2. Nella pagina del repository, clicca **Add file → Upload files** e trascina i tre file:
   `index.html`, `data.json`, `README_dashboard.md`.
3. Clicca **Commit changes**.
4. Vai su **Settings → Pages**. Sotto "Build and deployment", scegli **Deploy from a branch**,
   branch `main`, cartella `/ (root)`. Salva.
5. Dopo circa un minuto GitHub mostra l'indirizzo pubblico. Quello è il link permanente,
   consultabile da telefono.

## Aggiornamento mensile

1. Carica i nuovi PDF su Google Drive come al solito.
2. Chiedimi di aggiornare il dataset (in questa chat o in una nuova: il progetto mantiene il
   contesto tra le sessioni).
3. Ti consegno un `data.json` aggiornato.
4. Sul repository, apri `data.json`, clicca l'icona della matita (Edit), cancella tutto,
   incolla il nuovo contenuto e fai **Commit changes**. In alternativa **Add file → Upload
   files**, trascina il nuovo `data.json` e conferma la sovrascrittura.
5. La dashboard si aggiorna da sola entro un minuto. Non serve toccare `index.html`.

Ogni commit resta nello storico: dal *History* di `data.json` puoi sempre vedere come sono
cambiati i numeri da un aggiornamento all'altro.

## Struttura di `data.json`

```json
{
  "luce_monthly": [ … ],
  "gas_monthly":  [ … ],
  "gas_conguaglio_note": [ … ],
  "acqua_periodi": [ … ],
  "note_rilevanti": [ … ],
  "meta": { "aggiornato": "2026-07-17", "note": "…", "note_rilevanti_regola": "…" }
}
```

### `luce_monthly` e `gas_monthly`

Una riga per mese, in ordine cronologico. La dashboard non riordina né completa i buchi:
quello che c'è nell'array è quello che finisce nei grafici.

```json
{
  "month": "2026-06",
  "f": "Octopus",
  "c": 86,
  "mat": 16.17,
  "tra": 9.12,
  "one": 2.60,
  "tot": 30.68,
  "f1": 36,
  "f23": 50,
  "mat_var_unit": 0.098372,
  "note": ""
}
```

| Campo | Significato |
|---|---|
| `month` | mese, formato `AAAA-MM` |
| `f` | fornitore: `SEN`, `Eni` o `Octopus` |
| `c` | consumo del mese (kWh per la luce, Smc per il gas) |
| `tot` | totale della bolletta attribuito al mese (EUR). Per la luce è al netto del canone RAI |
| `mat` | materia prima, quota fissa inclusa (EUR) |
| `tra` | trasporto e gestione rete (EUR) |
| `one` | oneri di sistema (EUR) — può essere negativo |
| `mat_var_unit` | prezzo della **sola quota variabile** della materia prima (EUR/kWh o EUR/Smc) |
| `f1` / `f23` | consumo per fascia oraria in kWh — **solo luce** |
| `note` | testo libero, non mostrato in pagina: serve a te e a me per ricostruire la fonte |
| `ripartito` | `true` se il mese non nasce da una lettura reale (vedi sotto). Se assente vale `false` |

Le voci `mat`, `tra` e `one` non sommano a `tot`: la differenza è la riga
"Imposte e IVA" dei grafici, che la dashboard calcola come `tot − mat − tra − one`.

Un valore `null` significa "dato non disponibile per quel mese": nei grafici diventa un buco,
nelle tabelle un trattino.

**Il flag `ripartito`.** Quando il contatore non è stato letto e il costo del mese nasce da una
ripartizione (pro-rata sui giorni o sul consumo), la riga porta `"ripartito": true`. La
dashboard disegna quei mesi con punto vuoto e linea tratteggiata — barra sbiadita con bordo nel
grafico dei consumi — e lo dichiara in legenda. Serve a non farli leggere come misure reali:
è affidabile il totale del periodo, non il singolo mese. Oggi il flag è sui sei mesi gas
ago 25 – gen 26, coperti da un'unica lettura reale di cessazione.

### `note_rilevanti`

Le segnalazioni che compaiono nel riquadro "Da tenere d'occhio", in cima alla pagina.

```json
{
  "data": "2026-03-10",
  "tipo": "anomalia",
  "ambito": "Gas",
  "titolo": "Consumo gas ago25-gen26 quasi triplo rispetto al fatturato in acconto",
  "testo": "…"
}
```

- `data` è la data dell'**evento segnalato**, non quella in cui la nota è stata scritta.
- `tipo`: `rincaro`, `anomalia`, `dati_mancanti` o `info`. Determina colore ed etichetta.
- `ambito`: `Luce`, `Gas`, `Acqua` o `Entrambe`. Filtra la nota sulla scheda corrispondente.

Di default la pagina mostra solo gli ultimi 12 mesi; l'interruttore "Tutte" apre lo storico
completo.

### `acqua_periodi`

Una riga per bolletta, in ordine cronologico. L'acqua (Barbieri Edi/Publiacqua) fattura a trimestre, non a
mese: ogni riga è un intero periodo di fatturazione (`"AAAA-PN"`, N da 1 a 4), non un mese. Livello di
dettaglio "massimo" nel senso che ogni campo che compare in bolletta ha una colonna propria, comprese le voci
una tantum (deposito cauzionale) e la provenienza delle letture.

```json
{
  "periodo": "2026-P2",
  "f": "Publiacqua",
  "data_emissione": "2026-07-02",
  "giorni": 94,
  "lettura_prec": 51,
  "lettura_prec_data": "2026-03-20",
  "lettura_att": 54,
  "lettura_att_data": "2026-06-22",
  "tipo_lettura": "utente",
  "consumo_rilevato": 3,
  "conguaglio": null,
  "c": 3,
  "acqua_importo": 13.44,
  "quota_fissa": 15.62,
  "addebiti_accrediti": null,
  "servizi": 10.00,
  "arrotondamento": -0.01,
  "iva": 2.20,
  "tot": 41.25,
  "note": ""
}
```

| Campo | Significato |
|---|---|
| `periodo` | periodo di fatturazione, formato `AAAA-PN` (1°-4° periodo dell'anno) |
| `f` | gestore/ente fornitore: sempre `Publiacqua` in questa serie (fatturato da Barbieri Edi) |
| `data_emissione` | data di emissione della bolletta |
| `giorni` | giorni coperti dal periodo (variabile, il trimestre non è mai esatto) |
| `lettura_prec` / `lettura_att` | valori del contatore a inizio e fine periodo, con le rispettive date |
| `tipo_lettura` | come è stata presa la lettura finale: `utente` (reale), `credito` (nessuna lettura, a credito), `presunta` (stimata), `cambio contatore` |
| `consumo_rilevato` | mc dalla sola differenza di lettura del periodo |
| `conguaglio` | mc extra conguagliati da letture stimate/a credito precedenti, se presenti |
| `c` | mc **fatturati** nel periodo ("Totale MC" in bolletta) — questo è il campo usato in tutti i grafici di consumo |
| `acqua_importo` | "TOTALE IMPORTO ACQUA": consumo a tariffa + compensazione tariffe (EUR) |
| `quota_fissa` | "TOTALE QUOTA FISSA ENTE FORNITORE SERVIZIO IDRICO", prorata sui giorni (EUR) |
| `addebiti_accrediti` | voci una tantum indipendenti dal consumo (tipicamente deposito cauzionale), `null` se assenti |
| `servizi` | "TOTALE SERVIZIO" (spese SEPA + servizio), soggetto a IVA 22% (EUR) |
| `arrotondamento` | somma degli arrotondamenti precedente/attuale di bolletta (EUR) |
| `iva` | imposta IVA 22% sulla sola voce `servizi` (EUR) |
| `tot` | "TOTALE BOLLETTA" / "TOTALE DA PAGARE" — deve sempre coincidere con `acqua_importo + quota_fissa + (addebiti_accrediti||0) + servizi + arrotondamento + iva`, verificato riga per riga |
| `note` | testo libero: segnala letture non reali, cambio contatore, movimenti di deposito |

I periodi dal 4° 2021 al 4° 2022 hanno `tipo_lettura` `credito` o `presunta`: il gestore non ha mai letto il
contatore in quella finestra, il consumo fatturato è solo il conguaglio. La dashboard li disegna con punto
vuoto/tratteggio e barre sbiadite, come i mesi "ripartiti" di luce e gas.

### `gas_conguaglio_note`

Archivio delle due bollette di ricalcolo Eni (76,37 € e 60,16 €). **Non è più usato dalla
dashboard**: dal 17/07/2026 quegli importi sono confluiti nella serie `gas_monthly`, quindi
lasciarli anche qui li conterebbe due volte. Resta nel file come traccia documentale.

### `meta`

`aggiornato` è la data mostrata in testa alla pagina. `note` e `note_rilevanti_regola` sono
promemoria per me: non compaiono nell'interfaccia.

## Cosa c'è nella pagina

- **Pannello di sintesi** — spesa luce+gas+acqua degli ultimi 12 mesi, confronto con i 12 precedenti,
  e come si divide tra le tre utenze (per l'acqua, trimestrale, "ultimi 12 mesi" è approssimato con gli
  ultimi 4 periodi).
- **Quattro riquadri per utenza** — ultima bolletta, spesa e consumo a 12 mesi (o 4 periodi per l'acqua),
  prezzo attuale, ciascuno con la variazione e l'andamento recente.
- **Da tenere d'occhio** — le segnalazioni sulle bollette nuove.
- **Le sezioni di dettaglio** — prezzo, spesa, consumo, prezzo pieno, composizione del costo,
  fasce orarie (luce) o verifica del ricalcolo (gas), confronto fornitori, riepiloghi annuali.
  Ogni sezione ha un interruttore per cambiare vista — il confronto per stagione si apre per
  primo (è quello che risponde alla domanda più comune: "come sto messo rispetto all'anno
  scorso?"), poi l'andamento mese per mese. Le fasce orarie si aprono sulla vista in percentuale.
  Un "Come si legge" sotto ogni grafico spiega fonte e formula.
- **Da dove arrivano questi numeri** — la metodologia completa, in fondo a ciascuna scheda.

Tema chiaro e scuro: segue le impostazioni del telefono, con interruttore in alto a destra.

## Note tecniche

- Nessuna build, nessuna dipendenza da installare. Chart.js e i font arrivano da CDN.
- Le animazioni dei grafici sono disattivate di proposito. Con le animazioni attive Chart.js
  disegna solo dentro `requestAnimationFrame`: se la pagina si apre in una scheda in secondo
  piano, o il telefono è in risparmio energetico, quel frame non arriva mai e i grafici restano
  vuoti. Senza animazione il disegno è sincrono e non dipende da nulla.
- La tavolozza dei grafici è verificata per contrasto e daltonismo (protanopia/deuteranopia).
- Nei confronti stagionali ogni anno ha un colore fisso e ben distinto dagli altri — non una
  scala di uno stesso colore. Cinque tinte (arancio, verde, azzurro, viola, magenta), scelte
  cercando sistematicamente sulla ruota dei colori quelle che superano i test di separazione a
  tutte le coppie possibili, sia in tema chiaro sia scuro, con lo stesso hex in entrambi. Il
  colore segue la posizione dell'anno nell'elenco cronologico: quando un nuovo anno si aggiunge
  in coda, gli anni precedenti non cambiano colore. Il quinto colore è di scorta per quando
  comparirà un quinto anno nel dataset (oggi il dataset ne copre quattro, 2023–2026); se in
  futuro se ne aggiungesse un sesto andrebbe ripetuta la ricerca, perché nessuna sesta tinta a
  quella luminosità supera la stessa soglia di separazione.
- I grafici a colonna (composizione, consumo, fasce orarie) hanno oltre 40 mesi in ascissa:
  su schermo stretto le barre diventerebbero una riga di pixel illeggibile. Il riquadro che li
  contiene scorre in orizzontale, e il grafico dentro mantiene una larghezza minima leggibile
  (820px); su schermi larghi il riquadro è già più largo di così, quindi lì non cambia nulla e
  lo scroll non compare.

## File

| File | Contenuto |
|---|---|
| `index.html` | La dashboard. Nessuna build: si apre così com'è |
| `data.json` | Il dataset — l'unico file da toccare per gli aggiornamenti mensili |
| `README_dashboard.md` | Questo file |
