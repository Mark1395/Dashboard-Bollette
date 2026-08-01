# Dashboard bollette luce e gas

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
- `ambito`: `Luce`, `Gas` o `Entrambe`. Filtra la nota sulla scheda corrispondente.

Di default la pagina mostra solo gli ultimi 12 mesi; l'interruttore "Tutte" apre lo storico
completo.

### `gas_conguaglio_note`

Archivio delle due bollette di ricalcolo Eni (76,37 € e 60,16 €). **Non è più usato dalla
dashboard**: dal 17/07/2026 quegli importi sono confluiti nella serie `gas_monthly`, quindi
lasciarli anche qui li conterebbe due volte. Resta nel file come traccia documentale.

### `meta`

`aggiornato` è la data mostrata in testa alla pagina. `note` e `note_rilevanti_regola` sono
promemoria per me: non compaiono nell'interfaccia.

## Cosa c'è nella pagina

- **Pannello di sintesi** — spesa luce+gas degli ultimi 12 mesi, confronto con i 12 precedenti,
  e come si divide tra le due utenze.
- **Quattro riquadri per utenza** — ultima bolletta, spesa e consumo a 12 mesi, prezzo attuale,
  ciascuno con la variazione e l'andamento recente.
- **Da tenere d'occhio** — le segnalazioni sulle bollette nuove.
- **Le sezioni di dettaglio** — prezzo, spesa, consumo, prezzo pieno, composizione del costo,
  fasce orarie (luce) o verifica del ricalcolo (gas), confronto fornitori, riepiloghi annuali.
  Ogni sezione
  ha un interruttore per cambiare vista (nel tempo / per stagione, colonne / linee / percentuali)
  e un "Come si legge" che spiega fonte e formula.
- **Da dove arrivano questi numeri** — la metodologia completa, in fondo a ciascuna scheda.

Tema chiaro e scuro: segue le impostazioni del telefono, con interruttore in alto a destra.

## Note tecniche

- Nessuna build, nessuna dipendenza da installare. Chart.js e i font arrivano da CDN.
- Le animazioni dei grafici sono disattivate di proposito. Con le animazioni attive Chart.js
  disegna solo dentro `requestAnimationFrame`: se la pagina si apre in una scheda in secondo
  piano, o il telefono è in risparmio energetico, quel frame non arriva mai e i grafici restano
  vuoti. Senza animazione il disegno è sincrono e non dipende da nulla.
- La tavolozza dei grafici è verificata per contrasto e daltonismo (protanopia/deuteranopia).
- Nei confronti stagionali gli anni **non** hanno un colore ciascuno: nessuna quaterna della
  tavolozza supera i test di separazione a tutte le coppie sulla superficie scura, e quattro
  linee sovrapposte diventavano illeggibili. Un anno alla volta prende il colore dell'utenza
  (ambra per la luce, blu per il gas) e gli altri arretrano su una scala di grigi ordinata, dal
  più vecchio — il più sbiadito — al più recente. L'anno in evidenza si sceglie con i pulsanti
  sopra al grafico, e la scelta vale per tutti i confronti stagionali di entrambe le utenze.

## File

| File | Contenuto |
|---|---|
| `index.html` | La dashboard. Nessuna build: si apre così com'è |
| `data.json` | Il dataset — l'unico file da toccare per gli aggiornamenti mensili |
| `README_dashboard.md` | Questo file |
