# Popup chiusura per ferie di agosto — Design

## Contesto

Iglesias Camper Service è chiusa per ferie dal 1° al 31 agosto e durante questo
periodo può garantire solo supporto telefonico. Va comunicato ai visitatori
della home page (IT: `index.html`, EN: `en/index.html`) con un popup in stile
alert, ma senza che il testo dell'avviso finisca nel sorgente HTML statico
scaricato dai crawler (per non essere indicizzato/cacheato).

## Obiettivo

Mostrare, poco dopo il caricamento della pagina, un popup modale che informa
gli utenti della chiusura di agosto e offre una CTA per chiamare. Il popup:

- non deve esistere come markup nel sorgente HTML statico (generato interamente via JS)
- deve bloccare visivamente il contenuto sottostante (overlay) finché non viene chiuso
- una volta chiuso, non deve ripresentarsi (persistenza via `localStorage`)
- deve essere presente, con testo localizzato, sia in `index.html` (IT) sia in `en/index.html` (EN)

## Approccio

**Generazione 100% via JS.** Nessun markup del popup nell'HTML statico.
Un IIFE nello script di fondo pagina, eseguito dopo l'evento `load` con un
breve ritardo (~1s), costruisce overlay e card via `document.createElement` /
`textContent` e li appende al `<body>`. Le uniche tracce statiche sono le
regole CSS nel blocco `<style>` esistente (nessun testo, solo classi).

Scartata l'alternativa "HTML nascosto con `display:none` + classe `.show`"
(pattern già usato da `#lang-banner`/`#cookie-banner`) perché il testo
resterebbe comunque leggibile nel sorgente HTML grezzo, contro il requisito
esplicito di non farlo leggere ai crawler.

## Architettura

Aggiunta di un nuovo blocco IIFE in coda allo script esistente, sia in
`index.html` sia in `en/index.html` (stessa logica, stringhe localizzate
inline in ciascun file, coerentemente con come le due pagine sono già
duplicate/tradotte indipendentemente nel resto del sito).

```
window.addEventListener('load', () => {
  setTimeout(() => {
    if (localStorage.getItem('oc-august-notice-dismissed')) return;
    // costruzione DOM overlay + card via createElement
  }, 1000);
});
```

Chiave `localStorage` condivisa (`oc-august-notice-dismissed`) tra IT ed EN,
essendo stesso dominio/origine: se l'utente chiude il popup su una lingua,
resta chiuso anche passando all'altra.

Chiusura possibile tramite: bottone "×", click sull'overlay (fuori dalla
card), tasto `Escape`. Ognuno di questi imposta il flag in `localStorage` e
rimuove gli elementi dal DOM.

## Markup generato (equivalente logico)

```html
<div class="oc-modal-overlay" role="presentation">
  <div class="oc-modal-card" role="dialog" aria-modal="true" aria-label="…">
    <button class="oc-modal-close" aria-label="Chiudi">×</button>
    <h2>Chiusi per ferie</h2>
    <p>Saremo chiusi dal 1° al 31 agosto. In questo periodo possiamo
       garantire solo supporto telefonico.</p>
    <a href="tel:+393513531335" data-cta="tel_august_popup" class="oc-modal-cta">
      📞 Chiama ora
    </a>
  </div>
</div>
```

Testi EN corrispondenti in `en/index.html`:
- Titolo: "Closed for summer break"
- Corpo: "We're closed from August 1 to 31. During this period we can only
  offer phone support."
- CTA: "📞 Call now"

## Stile visivo

Nuove classi CSS aggiunte al blocco `<style>` esistente (nessun testo, solo
regole), coerenti con la palette del sito (`--c-cream`, `--c-amber`,
`--c-forest-dk`, ecc.) e con lo stile già usato da `#cookie-banner` /
`.btn-cookie-solid`:

- `.oc-modal-overlay`: `position: fixed; inset: 0; background: rgba(0,0,0,.55); z-index: 10010;` centra la card con flexbox
- `.oc-modal-card`: sfondo `--c-cream`, bordo/accento `--c-amber` (es. `border-top: 4px solid var(--c-amber)`), padding generoso, border-radius, box-shadow, `max-width` ~380px, animazione di ingresso leggera (fade + scale) coerente con `.fade-up` già presente nel sito
- `.oc-modal-close`: bottone discreto in alto a destra, stile simile a `#lang-banner-close`
- `.oc-modal-cta`: bottone pieno amber, stile simile a `.btn-cookie-solid`

## Interazione e accessibilità

- Al momento dell'apertura, il focus viene spostato sulla card (o sul bottone
  di chiusura)
- `Escape` chiude il popup
- Click sull'overlay (fuori dalla card) chiude il popup
- `role="dialog"` + `aria-modal="true"` + `aria-label` sulla card
- Non viene implementato un vero focus-trap (fuori scope per un avviso
  semplice con un solo link/bottone interattivo oltre alla chiusura)

## Tracking

Il link "Chiama ora"/"Call now" ha `data-cta="tel_august_popup"`, già
intercettato dal listener GA delegato esistente
(`document.addEventListener('click', …)` a fondo pagina) — nessun codice di
tracking aggiuntivo necessario.

## Rimozione a fine periodo

Il popup non ha logica di data automatica: rimane attivo finché il codice
non viene rimosso manualmente dai due file HTML a fine agosto. Non in scope
automatizzare la rimozione.

## Fuori scope

- Nessuna verifica automatica della data corrente (il popup non si
  autodisattiva dopo il 31 agosto)
- Nessun focus-trap completo
- Nessuna modifica a sitemap/robots/meta tag
