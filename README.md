# Brain Vita

Il solitario con i pioli, in versione minimal. PWA statica: un solo `index.html` (HTML, CSS e JS inline), nessuna build.

- Tavola inglese (33 buchi) o francese (37)
- Annulla, pausa (automatica quando si esce dall'app), record salvati in locale
- Quattro temi, guida animata, funziona offline

## Sviluppo

```bash
python3 -m http.server 8765
```

Poi apri http://localhost:8765. Il service worker si registra solo su http(s).

## Rilascio

Il sito è su Netlify, senza build: la cartella radice viene pubblicata così com'è.
A ogni rilascio aumenta `VERSION` in `sw.js` per aggiornare la cache.
