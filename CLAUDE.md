# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Progetto

Brain Vita: solitario con i pioli come PWA statica. Niente build, niente dipendenze, niente test né linter. Tutto il gioco (HTML, CSS, JS) sta inline in `index.html`; `sw.js` e `manifest.webmanifest` completano la PWA. Mantieni questa struttura (un solo `index.html`) salvo richiesta contraria. UI e commenti sono in italiano.

## Comandi

- Anteprima locale: `python3 -m http.server 8765` dalla radice, poi http://localhost:8765. Il service worker si registra solo su http(s), non da `file://`.
- Rilascio: push su `main` (GitHub `fracarbo/brainvita`); Netlify (sito `beamish-fox-4cf5ca`, site ID `89018077-82a0-4774-a396-3116abe09d3d`) pubblica la radice così com'è, vedi `netlify.toml`. Non creare un nuovo sito Netlify.
- Non cambiare impostazioni di account o sito (Netlify, DNS, domini) senza ok esplicito.

## Regole da ricordare

- **A ogni modifica ai file serviti aumenta `VERSION` in `sw.js`** (`bv-vN`): è ciò che forza l'aggiornamento della cache. Il SW fa stale-while-revalidate, quindi dopo un deploy serve a volte un secondo ricaricamento per vedere la versione nuova.
- Se aggiungi un file alla shell offline, inseriscilo anche in `SHELL` in `sw.js`.
- Il repo è pubblico: i commit usano l'email noreply di GitHub (config locale del repo), non quella personale.
- Il badge "Powered by Netlify" è già disattivato nelle impostazioni Netlify. Il layout lascia comunque ~56px in fondo a `.app` (`--badge-space`).
- Il blocco `window.claude.hot` in `index.html` serve solo all'anteprima degli artifact di Claude; è innocuo e rimovibile.

## Architettura di `index.html`

Un unico IIFE; stato in `S` (variabile di modulo) e render dichiarativo con `render()` che riscrive DOM da `S`.

- **Tavola**: griglia 7×7 piatta (`N = 7`), indice `i = r*N + c`, `CENTER = 24`. Celle: `-1` fuori tavola, `0` vuoto, `1` piolo. `VALID.en` (33 buchi) e `VALID.fr` (37) definiscono le forme. Le 49 celle sono bottoni creati una volta da `build()`; `render()` ne aggiorna classi (`void/on/sel/target`).
- **Mosse**: `movesFrom(i)` calcola i salti a distanza 2 in linea retta; `jump()` aggiorna `S.board`/`S.history`; `undo()` ripristina dalla cronologia; `finish()` scatta quando `anyMove()` è falso e salva il record.
- **Persistenza** (`localStorage`, sempre dentro `store()` con try/catch): `brainvita-best-en`, `brainvita-best-fr`, `brainvita-skin`. La partita in corso non è persistita.
- **Temi**: `SKINS` + variabili CSS su `html[data-skin]`; `render()` aggiorna anche `meta[name=theme-color]`.
- **Tempo e pausa**: il cronometro è `S.startedAt` (timestamp) o `S.elapsed` (secondi, quando fermo). Fermarlo = `S.elapsed = elapsed(); S.startedAt = null`; riprendere = `S.startedAt = Date.now() - S.elapsed*1000`. Tre meccanismi lo toccano, e vanno tenuti coerenti: pausa manuale (`setPaused`, flag `S.paused`, overlay `#paused`, tavola `inert`), pausa automatica su `visibilitychange` (solo se `S.startedAt` è attivo), e la sospensione temporanea mentre sono aperte le finestre di guida e di conferma (flag locali `paused`/`cPaused`). Se tocchi uno, controlla gli altri.
- **Conferme**: `confirmThen()` mostra `#confirm` per Ricomincia e per il cambio 33/37, ma solo con partita in corso (`S.moves > 0` e non `S.over`); altrimenti esegue subito.
- **Guida**: `<dialog id="howto">` con 3 passi; animazioni solo CSS (keyframes `d*`) su SVG inline. La mini tavola del passo 3 è generata da `setupHowTo()`. Con `prefers-reduced-motion` le animazioni sono disattivate globalmente.

## Trappole note

- Negli elementi SVG la proprietà `.hidden` non funziona: usa `toggleAttribute('hidden', …)` (e `.btn svg[hidden]{display:none}` in CSS).
- Il browser di anteprima di Claude Code non avvia `python3 -m http.server` da `launch.json` (PermissionError su `os.getcwd()`): avvialo con Bash in background e apri l'URL con `preview_start` e `url`.
