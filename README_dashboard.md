# Dashboard Bollette Luce e Gas

Dashboard statica (HTML + Chart.js) che legge i dati da `data.json`. Pensata per GitHub Pages:
aggiornare i dati ogni mese significa sostituire un solo file, senza toccare `index.html`.

## Setup iniziale (una tantum, ~5 minuti)

1. Vai su [github.com/new](https://github.com/new) e crea un repository. Puoi chiamarlo ad es.
   `bollette-dashboard`. Pubblico va bene: **nessun dato personale è presente** in questi file
   (niente nome, indirizzo, POD/PDR, codice fiscale o numero cliente — solo periodo, fornitore,
   consumi e costi).
2. Nella pagina del repository appena creato, clicca **Add file → Upload files** e trascina questi
   tre file: `index.html`, `data.json`, `README.md`.
3. Clicca **Commit changes**.
4. Vai su **Settings → Pages**. Sotto "Build and deployment", scegli **Deploy from a branch**,
   branch `main`, cartella `/ (root)`. Salva.
5. Dopo circa un minuto, GitHub mostra l'indirizzo pubblico (di solito
   `https://TUO-USERNAME.github.io/bollette-dashboard/`). Quello è il link permanente della
   dashboard, consultabile da telefono.

## Aggiornamento mensile

Ogni mese, quando arrivano nuove bollette:

1. Carica i nuovi PDF su Google Drive come al solito.
2. Chiedimi di aggiornare il dataset (in questa chat o in una nuova, il progetto mantiene il
   contesto tra le sessioni).
3. Ti consegnerò un nuovo `data.json` aggiornato.
4. Sul repository GitHub, apri `data.json`, clicca l'icona della matita (Edit), cancella tutto
   il contenuto, incolla quello nuovo, poi **Commit changes**.
   (In alternativa: **Add file → Upload files**, trascina il nuovo `data.json` e conferma la
   sovrascrittura.)
5. La dashboard pubblicata si aggiorna da sola entro un minuto — non serve toccare `index.html`
   né rifare il deploy.

Ogni commit resta nello storico del repository: puoi sempre tornare indietro e vedere esattamente
come sono cambiati i numeri aggiornamento dopo aggiornamento (History del file `data.json`).

## Struttura di data.json

```json
{
  "luce": {
    "main": [ {"p":"Lug23","f":"SEN","c":188,"tot":57.05,"mat":30.85,"tra":15.47,"one":5.54,"f1":70,"f23":118,"note":""}, ... ],
    "conguaglio": [ ... ]
  },
  "gas": { "main": [...], "conguaglio": [...] },
  "meta": { "aggiornato": "2026-07-13", "note": "..." }
}
```

- `p` = periodo, `f` = fornitore, `c` = consumo (kWh o Smc), `tot` = costo totale (EUR, luce al
  netto del canone RAI), `mat`/`tra`/`one` = materia prima / trasporto-rete / oneri di sistema,
  `f1`/`f23` = fascia oraria (solo luce), `note` = testo libero.
- Le bollette in `conguaglio` sono escluse dai grafici di trend (correzioni retroattive, non
  rappresentano un consumo di periodo).
- Valori `null` = dato non disponibile per quella riga (i grafici lo gestiscono come "gap").

## File

| File | Contenuto |
|---|---|
| `index.html` | Dashboard (nessuna dipendenza da build, apri/carica così com'è) |
| `data.json` | Dataset — è l'unico file da toccare per gli aggiornamenti mensili |
| `README.md` | Questo file |
