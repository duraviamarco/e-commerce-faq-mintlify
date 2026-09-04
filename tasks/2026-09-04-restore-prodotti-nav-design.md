# Design: ripristino "Prodotti e Schede Sicurezza" + nav a 6 gruppi + sblocco deploy

Data: 2026-09-04
Deployment: `3-dz-spa` (Mintlify) — repo `mintlify-community-2/docs-3-dz-spa-d2c306cf`, deploy branch `main`.

## Contesto

- `main` (HEAD `c6cfe91`) non fa deploy da ~21h: `Invalid docs.json: #.redirects: Each source path can only be used once`.
- `main` ha inoltre: sezione "Prodotti e Schede Sicurezza" cancellata (cartella `prodotti/` vuota, gruppo nav `schede-sicurezza` vuoto), nessuna pagina `problemi-tecnici/index`, homepage con link rotti (`/informazioni-generali/panoramica`, `/preventivi/richiedere-un-preventivo`).
- Il branch `admin-mcp/restore-prodotti-schede-sicurezza-f2beae6` (base `main` vecchio `0e00eab`) contiene uno stato completo e coerente: 6 gruppi nav, `docs.json` valido, tutte le pagine `prodotti/*` con contenuto reale, `problemi-tecnici/index.mdx`. Usato come **sorgente dei contenuti**, non come base del merge.

## Decisioni approvate

1. **Approccio**: ricostruire lo stato target su un branch nuovo creato da `main` attuale (non promuovere `f2beae6`, per evitare conflitti/regressioni).
2. **Nav**: `navigation.tabs` → un tab "Centro Assistenza" con 6 gruppi collassabili (`collapsed: true`): Home, Informazioni Generali, Ordini/Spedizioni e Resi, Preventivi, **Prodotti e Schede Sicurezza**, Problemi Tecnici.
3. **Pagine "I miei prodotti" duplicate** (`informazioni-generali/sezione-i-miei-prodotti` e `ordini/sezione-i-miei-prodotti`): **tenute entrambe**.
4. **Snippet `/snippets/Header3DZ.mdx` e `/snippets/Footer3DZ.mdx`** referenziati dalle pagine prodotti in `f2beae6`: **rimossi** (import + tag) dalle pagine ricreate, per uniformità col resto del sito che non li usa.
5. **Redirects**: `docs.json` riscritto pulito; rimossi i redirect `/prodotti/*` (le pagine tornano a esistere).
6. **Homepage**: fix dei 2 link rotti, card allineate alle 5 sezioni.
7. **Pulizia**: rimosso `problemi-tecnici/b.mdx` (file di test).
8. **Fuori scope**: pulizia dei ~68 branch `admin-mcp/*` residui — nessun tool disponibile per cancellare branch remoti (repo non accessibile dal token GitHub configurato); da fare manualmente su GitHub.

## Processo di implementazione

1. Checkout di lettura su `f2beae6` → copia contenuto delle 9 pagine, verifica esistenza `snippets/`.
2. Checkout nuovo branch da `main`.
3. `write_page` delle 9 pagine (snippet rimossi) · `update_config` per `docs.json` (nav + redirects) · fix `index.mdx` · `delete_node` di `b.mdx`.
4. `diff` di revisione → `save` in modalità PR (no auto-merge).
5. Verifica che il check di deploy Mintlify sul PR sia verde.
6. Review utente della preview PR → merge → verifica deploy live.

## Rischi

- Altre pagine potrebbero usare `Header3DZ`/`Footer3DZ`: verificato prima del save che non si rompa nulla.
- Se il deploy fallisse ancora sui redirect dopo la pulizia, investigare redirect auto-generati da Mintlify per le pagine spostate/cancellate.
