# Bottone di condivisione con tracking per canale

## Contesto

Il sito (`index.html` IT, `en/index.html` EN) ha già un pattern di tracking CTA generico: qualunque elemento con `data-cta="..."` viene intercettato da un listener delegato su `document` che spara un evento GA4 `cta_click` con `cta_id`, `cta_text`, `cta_url`, `page_location` (vedi `index.html` sezione "Tracciamento click sulle CTA"). GA4 viene caricato solo dopo consenso cookie (`loadGA()`), quindi nessun evento parte prima del consenso.

L'header ha già un toggle lingua IT/EN in stile "pillola" (`.lang-toggle` / `.lang-toggle-item`) accanto al CTA WhatsApp. Vogliamo aggiungere, nello stesso gruppo di controlli, un bottone di condivisione della pagina corrente, con tracking per canale scelto.

**Vincolo noto**: la Web Share API nativa (`navigator.share`) non espone mai quale app l'utente ha scelto nel menu di sistema (limite di privacy del browser stesso). Instagram non ha un intent web ufficiale per condividere un link arbitrario. Per avere breakdown per canale servono link diretti dedicati (WhatsApp, Facebook, Email); tutto il resto (incl. Instagram) passa da un'opzione generica "Altro" che usa il native share o, in mancanza, copia il link.

## Obiettivo

Un bottone icona-condivisione in header che apre un piccolo menu con 4 canali, ciascuno tracciato distintamente in GA4 tramite il pattern `data-cta` esistente (nessun codice di tracking nuovo da scrivere).

## Componenti

### 1. Bottone trigger (header)

- Nuovo elemento nel gruppo controlli header, tra `.lang-toggle` e il CTA WhatsApp (o subito dopo il lang-toggle, dentro lo stesso contenitore flex).
- `<button>` icona-soltanto, stile pillola coerente con `.lang-toggle-item` (stesso sfondo translucido `rgba(255,255,255,.08)`, stesso raggio, stessa altezza).
- Icona SVG "share" classica (tre nodi collegati da linee), stroke `var(--c-forest-lt)`, ~18px, stessa famiglia di stroke-width/linecap delle altre icone SVG del sito (vedi icona accessibilità: `stroke-width="2" stroke-linecap="round" stroke-linejoin="round"`).
- `aria-label="Condividi questa pagina"` (IT) / `"Share this page"` (EN).
- `aria-haspopup="menu"` `aria-expanded` sincronizzato con lo stato aperto/chiuso.
- `data-cta="share_menu_open"` — così in GA4 è visibile anche il tasso di apertura del menu, separato dalla scelta del canale.

### 2. Menu a comparsa

- `<div class="share-menu" role="menu">` posizionato in `position:absolute` ancorato sotto il bottone trigger (il trigger è `position:relative` per fare da riferimento).
- Contenuto (icona + etichetta testuale per ciascuna voce, per chiarezza — a differenza del trigger che è icona-soltanto):
  - **WhatsApp** → `<a role="menuitem" href="https://wa.me/?text=<title> - <url>" target="_blank" rel="noopener" data-cta="share_whatsapp">`
  - **Facebook** → `<a role="menuitem" href="https://www.facebook.com/sharer/sharer.php?u=<url>" target="_blank" rel="noopener" data-cta="share_facebook">`
  - **Email** → `<a role="menuitem" href="mailto:?subject=<title>&body=<url>" data-cta="share_email">`
  - **Altro** → `<button role="menuitem" data-cta="share_other">` — vedi logica JS sotto
- `<url>` = `encodeURIComponent(location.href)` (valutato runtime, così riflette IT `/` o EN `/en/` correttamente); `<title>` = `encodeURIComponent(document.title)`.
- Stile: card con sfondo `var(--c-forest-dk)`, bordo/ombra leggera coerente con `#cookie-banner`/dropdown esistenti, angoli arrotondati, ogni voce `display:flex;align-items:center;gap:.5rem`, hover con leggero cambio sfondo.

### 3. Logica JS (IIFE dedicata, stile coerente col resto del file)

```js
(() => {
  const trigger = document.getElementById('share-trigger');
  const menu = document.getElementById('share-menu');
  const openMenu  = () => { menu.classList.add('show'); trigger.setAttribute('aria-expanded','true'); };
  const closeMenu = () => { menu.classList.remove('show'); trigger.setAttribute('aria-expanded','false'); };

  trigger.addEventListener('click', e => {
    e.stopPropagation();
    menu.classList.contains('show') ? closeMenu() : openMenu();
  });
  document.addEventListener('click', e => {
    if (!menu.contains(e.target) && e.target !== trigger) closeMenu();
  });
  document.addEventListener('keydown', e => { if (e.key === 'Escape') closeMenu(); });

  document.getElementById('share-other').addEventListener('click', async () => {
    const shareData = { title: document.title, url: location.href };
    if (navigator.share) {
      try { await navigator.share(shareData); } catch (_) { /* utente ha annullato, nessun errore mostrato */ }
    } else {
      try {
        await navigator.clipboard.writeText(location.href);
        showCopiedToast();
      } catch (_) { /* clipboard non disponibile: nessun fallback ulteriore, edge case trascurabile */ }
    }
    closeMenu();
  });
})();
```

- `showCopiedToast()`: piccolo elemento assoluto vicino al trigger con testo "Link copiato!" (IT) / "Link copied!" (EN), classe `.show` aggiunta per ~2s poi rimossa via `setTimeout`, transizione CSS opacity già presente come pattern nel sito (`#wa-float` usa lo stesso approccio opacity+transition).
- Nessun nuovo listener di tracking: i 4 `data-cta` sopra vengono già intercettati dal listener generico esistente (`document.addEventListener('click', e => { const cta = e.target.closest('[data-cta]'); ... })`), quindi il click su un `<a>` (WhatsApp/Facebook/Email, che naviga via `target=_blank` o `mailto:`) e sul `<button>` "Altro" sparano `cta_click` con `cta_id` distinto, prima/durante la navigazione.

### 4. Gestione errori / edge case

- Menu nativo annullato dall'utente → `navigator.share()` rigetta con `AbortError`: catturato e ignorato silenziosamente, nessun toast d'errore.
- `navigator.clipboard` non disponibile (contesto non sicuro o browser molto vecchio) → fallimento silenzioso, nessun fallback aggiuntivo (rischio trascurabile, sito servito su HTTPS).
- Menu deve chiudersi anche se l'utente clicca altrove in pagina o preme Esc (vedi sopra).

### 5. Duplicazione IT/EN

Stessa struttura in `index.html` ed `en/index.html`, uniche differenze:
- Etichette voci menu: IT "WhatsApp / Facebook / Email / Altro", EN "WhatsApp / Facebook / Email / More".
- `aria-label` bottone trigger e testo toast localizzati.
- `data-cta` values identici in entrambe le lingue (permette di aggregare i due canali in GA4 senza distinzione, dato che `page_location` già distingue IT da EN se serve un breakdown per lingua).

## Verifica

- Screenshot headless a 500px (come per le modifiche precedenti) per confermare che il nuovo bottone non causi overflow/taglio nell'header su mobile; se necessario, estendere la media query `@media (max-width:600px)` già esistente per adattare anche il nuovo elemento.
- Verifica manuale che ciascun link apra il canale corretto con URL e titolo codificati correttamente (test su IT e su EN).
- Verifica che il menu si chiuda su click esterno, su Esc, e dopo la scelta di un canale.
- Verifica che l'evento `cta_click` con `cta_id` corretto compaia in GA4 (via DebugView o Realtime) per ciascuna voce, coerente con quanto già verificato per le altre CTA del sito.

## Fuori scope

- Nessuna voce Instagram dedicata (vincolo tecnico, vedi Contesto).
- Nessuna nuova dimensione personalizzata GA4 da configurare in questa spec (già discusso in conversazione come attività separata per `cta_id`).
- Non tocca l'icona/bottone del pannello accessibilità (richiesta separata, gestita a parte).
