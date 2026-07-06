# Bottone di Condivisione — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Aggiungere in header un bottone icona-condivisione che apre un menu con WhatsApp/Facebook/Email/Altro, ciascuno tracciato in GA4 tramite il pattern `data-cta` già esistente nel sito.

**Architecture:** Sito statico a due pagine (`index.html` IT, `en/index.html` EN), nessun build step, nessun framework JS, nessun test runner. Ogni pagina ha CSS in un unico `<style>` in `<head>` e JS in un unico `<script>` prima di `</body>`. Le due pagine vengono mantenute in sincronia manualmente, modifica per modifica (pattern già usato in tutte le sessioni precedenti su questo repo).

**Tech Stack:** HTML/CSS/JS vanilla, Google Analytics 4 via `gtag`, nessuna dipendenza nuova.

## Global Constraints

- Non esiste un test runner in questo repo (nessun `package.json`, nessun Jest/Playwright). La "verifica" di ogni task è: (1) uno screenshot headless Chrome per il layout, (2) un controllo manuale interattivo con risultato atteso esplicito per il comportamento JS. Non introdurre Playwright/Puppeteer solo per questo — è fuori scope (YAGNI) per un sito statico senza altra tooling.
- Ogni modifica va replicata identica in `index.html` e `en/index.html`, con solo le stringhe di testo tradotte (mai la struttura, gli `id`, le classi o i `data-cta`).
- I valori `data-cta` sono già intercettati automaticamente dal listener generico in entrambi i file (cercare `// ── Tracciamento click sulle CTA`) — non aggiungere nessun nuovo codice di tracking, solo l'attributo `data-cta` sull'elemento giusto.
- Comandi di screenshot da usare per la verifica visiva (adatta il path se lo scratchpad è diverso):
  `"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu --hide-scrollbars --window-size=500,900 --screenshot=<output.png> "file://<percorso-assoluto>/index.html"`

---

## Task 1: CSS del bottone, del menu e del toast

**Files:**
- Modify: `index.html` (blocco `<style>`, subito dopo le regole `.lang-toggle-item.active`)
- Modify: `en/index.html` (stesso punto, stesso CSS — identico, nessuna traduzione nel CSS)

**Interfacce prodotte per i task successivi:**
- Classi: `.share-wrap` (contenitore posizionato, `position:relative`), `#share-trigger` (bottone icona), `.share-menu` / `.share-menu.show` (menu a comparsa), `.share-menu-item` (voce di menu), `.share-toast` / `.share-toast.show` (conferma copia link).

- [ ] **Step 1: Aggiungi il CSS in `index.html`**

Cerca questo blocco (già presente nel file):

```css
    a.lang-toggle-item:hover { opacity: .85; }
    .lang-toggle-item.active {
      background: var(--c-cream);
      color: var(--c-forest-dk);
      opacity: 1;
      cursor: default;
    }

    /* ── Header: prevent WhatsApp CTA overflow on narrow phones ── */
```

Sostituiscilo con (aggiunge il nuovo blocco CSS subito prima della media query esistente):

```css
    a.lang-toggle-item:hover { opacity: .85; }
    .lang-toggle-item.active {
      background: var(--c-cream);
      color: var(--c-forest-dk);
      opacity: 1;
      cursor: default;
    }

    /* ── Condivisione pagina (header) ─────────────────────── */
    .share-wrap { position: relative; flex-shrink: 0; }
    #share-trigger {
      display: flex; align-items: center; justify-content: center;
      width: 30px; height: 30px;
      background: rgba(255,255,255,.08);
      border: none; border-radius: 999px;
      cursor: pointer; padding: 0;
      transition: background-color .15s;
    }
    #share-trigger:hover { background: rgba(255,255,255,.18); }
    .share-menu {
      position: absolute;
      top: calc(100% + .5rem);
      right: 0;
      background: var(--c-forest-dk);
      border: 1px solid rgba(255,255,255,.12);
      border-radius: 12px;
      padding: .4rem;
      min-width: 175px;
      box-shadow: 0 8px 28px rgba(0,0,0,.35);
      display: none;
      flex-direction: column;
      gap: .1rem;
      z-index: 10001;
    }
    .share-menu.show { display: flex; }
    .share-menu-item {
      display: flex; align-items: center; gap: .55rem;
      padding: .5rem .6rem;
      border-radius: 8px;
      color: var(--c-cream);
      font-size: .82rem;
      font-weight: 600;
      text-decoration: none;
      background: none; border: none;
      cursor: pointer;
      text-align: left;
      width: 100%;
      font-family: inherit;
    }
    .share-menu-item:hover { background: rgba(255,255,255,.08); }
    .share-menu-item svg { flex-shrink: 0; }
    .share-toast {
      position: absolute;
      top: calc(100% + .5rem);
      right: 0;
      background: var(--c-forest-dk);
      color: var(--c-cream);
      font-size: .78rem;
      font-weight: 700;
      padding: .5rem .8rem;
      border-radius: 8px;
      box-shadow: 0 4px 18px rgba(0,0,0,.3);
      opacity: 0;
      pointer-events: none;
      transition: opacity .25s ease;
      white-space: nowrap;
      z-index: 10002;
    }
    .share-toast.show { opacity: 1; }

    /* ── Header: prevent WhatsApp CTA overflow on narrow phones ── */
```

- [ ] **Step 2: Applica lo stesso identico CSS in `en/index.html`**

Cerca lo stesso blocco (`a.lang-toggle-item:hover { opacity: .85; } ... /* ── Header: prevent WhatsApp CTA overflow on narrow phones ── */`) in `en/index.html` e applica la stessa sostituzione dello Step 1 (contenuto CSS identico, nessuna traduzione).

- [ ] **Step 3: Verifica visiva che il CSS non rompa nulla**

Run:
```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu --hide-scrollbars --window-size=1280,800 --screenshot=/tmp/task1-check.png "file:///Users/wepan/repo/officina-camper/index.html"
```
Expected: lo screenshot mostra l'header IT invariato (nessun elemento nuovo visibile ancora, perché l'HTML del bottone non è stato aggiunto in questo task — è normale). Nessun errore di rendering/layout rotto nel resto della pagina.

- [ ] **Step 4: Commit**

```bash
git add index.html en/index.html
git commit -m "Aggiunge CSS per il bottone di condivisione in header"
```

---

## Task 2: Markup HTML del bottone e del menu

**Files:**
- Modify: `index.html:551-554` (blocco `<!-- Language switcher -->`, subito dopo il tag di chiusura `</div>` del `.lang-toggle`)
- Modify: `en/index.html` (stesso punto)

**Interfacce consumate:** classi CSS definite nel Task 1 (`.share-wrap`, `#share-trigger`, `.share-menu`, `.share-menu-item`, `.share-toast`).

**Interfacce prodotte per il Task 3:** id `share-trigger`, `share-menu`, `share-whatsapp`, `share-facebook`, `share-email`, `share-other`, `share-toast` — il Task 3 li recupera con `document.getElementById(...)`.

- [ ] **Step 1: Inserisci il markup in `index.html`**

Cerca:

```html
      <!-- Language switcher -->
      <div class="lang-toggle" role="group" aria-label="Cambia lingua">
        <span class="lang-toggle-item active" aria-current="true">🇮🇹 IT</span>
        <a href="/en/" id="lang-switch-nav" data-cta="lang_switch_navbar" class="lang-toggle-item" aria-label="Switch to English">🇬🇧 EN</a>
      </div>

      <!-- CTA -->
```

Sostituiscilo con:

```html
      <!-- Language switcher -->
      <div class="lang-toggle" role="group" aria-label="Cambia lingua">
        <span class="lang-toggle-item active" aria-current="true">🇮🇹 IT</span>
        <a href="/en/" id="lang-switch-nav" data-cta="lang_switch_navbar" class="lang-toggle-item" aria-label="Switch to English">🇬🇧 EN</a>
      </div>

      <!-- Condividi -->
      <div class="share-wrap">
        <button id="share-trigger" type="button" aria-haspopup="menu" aria-expanded="false"
                aria-label="Condividi questa pagina" data-cta="share_menu_open">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="var(--c-forest-lt)"
               stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <circle cx="18" cy="5" r="3"/><circle cx="6" cy="12" r="3"/><circle cx="18" cy="19" r="3"/>
            <line x1="8.6" y1="10.6" x2="15.4" y2="6.4"/><line x1="8.6" y1="13.4" x2="15.4" y2="17.6"/>
          </svg>
        </button>
        <div id="share-menu" class="share-menu" role="menu" aria-label="Condividi tramite">
          <a id="share-whatsapp" role="menuitem" href="#" target="_blank" rel="noopener" data-cta="share_whatsapp" class="share-menu-item">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor" aria-hidden="true">
              <path d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.096 3.2 5.077 4.487.709.306 1.262.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347z"/>
              <path d="M12 0C5.373 0 0 5.373 0 12c0 2.136.562 4.14 1.54 5.877L0 24l6.29-1.513A11.94 11.94 0 0 0 12 24c6.627 0 12-5.373 12-12S18.627 0 12 0zm0 21.818a9.818 9.818 0 0 1-5.002-1.366l-.36-.214-3.733.898.938-3.63-.234-.374A9.818 9.818 0 1 1 12 21.818z"/>
            </svg>
            WhatsApp
          </a>
          <a id="share-facebook" role="menuitem" href="#" target="_blank" rel="noopener" data-cta="share_facebook" class="share-menu-item">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor" aria-hidden="true">
              <path d="M22 12a10 10 0 1 0-11.6 9.9v-7H7.9V12h2.5V9.8c0-2.5 1.5-3.9 3.8-3.9 1.1 0 2.2.2 2.2.2v2.4h-1.2c-1.2 0-1.6.8-1.6 1.6V12h2.8l-.4 2.9h-2.4v7A10 10 0 0 0 22 12z"/>
            </svg>
            Facebook
          </a>
          <a id="share-email" role="menuitem" href="#" data-cta="share_email" class="share-menu-item">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor"
                 stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <rect x="3" y="5" width="18" height="14" rx="2"/><path d="m3 7 9 6 9-6"/>
            </svg>
            Email
          </a>
          <button id="share-other" role="menuitem" type="button" data-cta="share_other" class="share-menu-item">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor"
                 stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <circle cx="18" cy="5" r="3"/><circle cx="6" cy="12" r="3"/><circle cx="18" cy="19" r="3"/>
              <line x1="8.6" y1="10.6" x2="15.4" y2="6.4"/><line x1="8.6" y1="13.4" x2="15.4" y2="17.6"/>
            </svg>
            Altro
          </button>
        </div>
        <span id="share-toast" class="share-toast">Link copiato!</span>
      </div>

      <!-- CTA -->
```

- [ ] **Step 2: Inserisci il markup tradotto in `en/index.html`**

Cerca lo stesso blocco `<!-- Language switcher -->` in `en/index.html` e applica la stessa sostituzione, con queste differenze testuali (struttura, id, classi e `data-cta` restano identici):
- `aria-label="Condividi questa pagina"` → `aria-label="Share this page"`
- `aria-label="Condividi tramite"` → `aria-label="Share via"`
- Etichetta voce menu `Altro` → `More`
- Testo `#share-toast` → `Link copied!`

- [ ] **Step 3: Verifica visiva (stato chiuso, mobile e desktop)**

Run:
```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu --hide-scrollbars --window-size=500,900 --screenshot=/tmp/task2-mobile.png "file:///Users/wepan/repo/officina-camper/index.html"
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu --hide-scrollbars --window-size=1280,800 --screenshot=/tmp/task2-desktop.png "file:///Users/wepan/repo/officina-camper/index.html"
```
Expected: in entrambi gli screenshot compare la nuova icona di condivisione (tre pallini collegati) tra il toggle lingua IT/EN e il bottone verde "Contattaci"/"Contact us", senza che nessun elemento dell'header venga tagliato o vada a capo. Il menu resta invisibile (perché `.share-menu` non ha ancora la classe `.show` e nessun JS la aggiunge finché non facciamo il Task 3).

Se nello screenshot mobile (500px) l'header risulta troppo stretto e qualcosa viene tagliato, apri `index.html`, cerca il blocco:
```css
    @media (max-width: 600px) {
      header .inner { gap: .6rem !important; }
      .brand-name { display: none; }
      a[data-cta="whatsapp_header"] { padding-left: .8rem !important; padding-right: .8rem !important; font-size: .7rem !important; }
    }
```
e aggiungi una riga `#share-trigger { width: 26px !important; height: 26px !important; }` dentro la stessa media query, in entrambi i file, poi rifai lo screenshot.

- [ ] **Step 4: Commit**

```bash
git add index.html en/index.html
git commit -m "Aggiunge markup del bottone e menu di condivisione in header"
```

---

## Task 3: Logica JS (apertura/chiusura menu, canali, fallback "Altro")

**Files:**
- Modify: `index.html` (blocco `<script>` finale, subito dopo il listener di `cookie-manage-link` e prima del commento `// ── Tracciamento click sulle CTA`)
- Modify: `en/index.html` (stesso punto)

**Interfacce consumate:** id HTML prodotti nel Task 2 (`share-trigger`, `share-menu`, `share-whatsapp`, `share-facebook`, `share-email`, `share-other`, `share-toast`).

- [ ] **Step 1: Aggiungi la logica JS in `index.html`**

Cerca:

```js
  document.getElementById('cookie-manage-link').addEventListener('click', e => {
    e.preventDefault();
    cookieBanner.classList.add('show');
  });

  // ── Tracciamento click sulle CTA (WhatsApp, telefono, Maps) ───
```

Sostituiscilo con:

```js
  document.getElementById('cookie-manage-link').addEventListener('click', e => {
    e.preventDefault();
    cookieBanner.classList.add('show');
  });

  // ── Condivisione pagina ────────────────────────────────────────
  (() => {
    const trigger = document.getElementById('share-trigger');
    const menu    = document.getElementById('share-menu');
    const toast   = document.getElementById('share-toast');

    const pageUrl   = encodeURIComponent(location.href);
    const pageTitle = encodeURIComponent(document.title);
    document.getElementById('share-whatsapp').href = `https://wa.me/?text=${encodeURIComponent(document.title + ' - ' + location.href)}`;
    document.getElementById('share-facebook').href = `https://www.facebook.com/sharer/sharer.php?u=${pageUrl}`;
    document.getElementById('share-email').href    = `mailto:?subject=${pageTitle}&body=${pageUrl}`;

    const openMenu  = () => { menu.classList.add('show');    trigger.setAttribute('aria-expanded', 'true'); };
    const closeMenu = () => { menu.classList.remove('show'); trigger.setAttribute('aria-expanded', 'false'); };

    trigger.addEventListener('click', e => {
      e.stopPropagation();
      menu.classList.contains('show') ? closeMenu() : openMenu();
    });
    document.addEventListener('click', e => {
      if (!menu.contains(e.target) && e.target !== trigger) closeMenu();
    });
    document.addEventListener('keydown', e => { if (e.key === 'Escape') closeMenu(); });
    menu.querySelectorAll('a').forEach(a => a.addEventListener('click', closeMenu));

    document.getElementById('share-other').addEventListener('click', async () => {
      const shareData = { title: document.title, url: location.href };
      if (navigator.share) {
        try { await navigator.share(shareData); } catch (_) { /* utente ha annullato */ }
      } else {
        try {
          await navigator.clipboard.writeText(location.href);
          toast.classList.add('show');
          setTimeout(() => toast.classList.remove('show'), 2000);
        } catch (_) { /* clipboard non disponibile, edge case trascurabile */ }
      }
      closeMenu();
    });
  })();

  // ── Tracciamento click sulle CTA (WhatsApp, telefono, Maps) ───
```

- [ ] **Step 2: Aggiungi la stessa logica in `en/index.html`**

Cerca lo stesso punto (`document.getElementById('cookie-manage-link')...` fino a `// ── Tracciamento click sulle CTA`) in `en/index.html` e applica la stessa sostituzione dello Step 1 — il codice JS è **identico** (nessuna stringa da tradurre: `document.title`/`location.href` prendono già i valori corretti della pagina EN a runtime).

- [ ] **Step 3: Verifica manuale interattiva (IT)**

Apri il file direttamente nel browser:
```bash
open /Users/wepan/repo/officina-camper/index.html
```
Esegui in ordine e verifica ogni risultato atteso:
1. Clicca l'icona di condivisione in header → **atteso:** si apre un menu con 4 voci (WhatsApp, Facebook, Email, Altro).
2. Premi `Esc` → **atteso:** il menu si chiude.
3. Riapri il menu, clicca da qualche parte fuori dal menu (es. sul titolo hero) → **atteso:** il menu si chiude.
4. Riapri il menu, clicca "WhatsApp" → **atteso:** si apre una nuova scheda `https://wa.me/?text=...` con il titolo della pagina e l'URL `file:///.../index.html` codificati nel testo.
5. Riapri il menu, clicca "Facebook" → **atteso:** si apre `https://www.facebook.com/sharer/sharer.php?u=...`.
6. Riapri il menu, clicca "Email" → **atteso:** si apre il client di posta con oggetto = titolo pagina e corpo = URL pagina.
7. Riapri il menu, clicca "Altro" → **atteso:** su Chrome/Safari desktop (nessun `navigator.share`) il link viene copiato negli appunti e appare per ~2 secondi il fumetto "Link copiato!"; su un browser/dispositivo con Web Share API si apre invece il menu nativo di condivisione del sistema.
8. Apri gli Strumenti per sviluppatori → Console e incolla `document.getElementById('share-whatsapp').href` → **atteso:** stringa che inizia con `https://wa.me/?text=`.

- [ ] **Step 4: Ripeti la stessa verifica manuale su `en/index.html`**

```bash
open /Users/wepan/repo/officina-camper/en/index.html
```
Stessi 8 controlli dello Step 3, con le etichette del menu in inglese (WhatsApp, Facebook, Email, More) e il fumetto "Link copied!".

- [ ] **Step 5: Verifica screenshot finale mobile (entrambe le lingue)**

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu --hide-scrollbars --window-size=500,900 --screenshot=/tmp/task3-it.png "file:///Users/wepan/repo/officina-camper/index.html"
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu --hide-scrollbars --window-size=500,900 --screenshot=/tmp/task3-en.png "file:///Users/wepan/repo/officina-camper/en/index.html"
```
Expected: header invariato rispetto al Task 2 (il JS non altera il layout quando il menu è chiuso).

- [ ] **Step 6: Commit e push**

```bash
git add index.html en/index.html
git commit -m "Aggiunge logica di apertura/chiusura menu e condivisione per canale"
git push
```

---

## Task 4: Voce Instagram nel menu di condivisione

**Contesto (aggiunto dopo l'approvazione della spec originale):** Instagram non ha un intent web ufficiale per condividere un link arbitrario (a differenza di WhatsApp/Facebook). Il workaround adottato: un voce di menu dedicata che copia il link negli appunti e mostra un fumetto con istruzioni ("incollalo su Instagram"), tracciata con un `data-cta` proprio così resta distinguibile in GA4 da "Altro".

**Files:**
- Modify: `index.html` (markup del menu, righe ~628-658 dell'header; JS del blocco "Condivisione pagina", righe ~1125-1163)
- Modify: `en/index.html` (stessi punti)

**Interfacce consumate:** id/classi prodotti nei Task 1-3 (`.share-menu`, `.share-menu-item`, `#share-toast`, `.share-toast`/`.share-toast.show`).
**Interfacce prodotte:** nuovo id `share-instagram` (bottone), nuova funzione di supporto `showToast(text)` che sostituisce l'uso diretto di `toast.classList.add('show')` nel blocco "Altro".

- [ ] **Step 1: Inserisci la voce di menu in `index.html`**

Cerca:

```html
          <a id="share-facebook" role="menuitem" href="#" target="_blank" rel="noopener" data-cta="share_facebook" class="share-menu-item">
```

Sostituiscilo con (aggiunge la voce Instagram subito prima di Facebook):

```html
          <button id="share-instagram" role="menuitem" type="button" data-cta="share_instagram" class="share-menu-item">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor"
                 stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <rect x="3" y="3" width="18" height="18" rx="5"/>
              <circle cx="12" cy="12" r="4"/>
              <circle cx="17.5" cy="6.5" r="1.1" fill="currentColor" stroke="none"/>
            </svg>
            Instagram
          </button>
          <a id="share-facebook" role="menuitem" href="#" target="_blank" rel="noopener" data-cta="share_facebook" class="share-menu-item">
```

- [ ] **Step 2: Applica lo stesso identico markup in `en/index.html`**

Stesso punto, stesso HTML esatto (l'etichetta "Instagram" è un nome di marchio, non va tradotta — resta invariata in entrambi i file, come già avviene per "WhatsApp" e "Facebook").

- [ ] **Step 3: Aggiorna il JS in `index.html`**

Cerca il blocco esistente:

```js
  // ── Condivisione pagina ────────────────────────────────────────
  (() => {
    const trigger = document.getElementById('share-trigger');
    const menu    = document.getElementById('share-menu');
    const toast   = document.getElementById('share-toast');

    const pageUrl   = encodeURIComponent(location.href);
    const pageTitle = encodeURIComponent(document.title);
    document.getElementById('share-whatsapp').href = `https://wa.me/?text=${encodeURIComponent(document.title + ' - ' + location.href)}`;
    document.getElementById('share-facebook').href = `https://www.facebook.com/sharer/sharer.php?u=${pageUrl}`;
    document.getElementById('share-email').href    = `mailto:?subject=${pageTitle}&body=${pageUrl}`;

    const openMenu  = () => { menu.classList.add('show');    trigger.setAttribute('aria-expanded', 'true'); };
    const closeMenu = () => { menu.classList.remove('show'); trigger.setAttribute('aria-expanded', 'false'); };

    trigger.addEventListener('click', e => {
      e.stopPropagation();
      menu.classList.contains('show') ? closeMenu() : openMenu();
    });
    document.addEventListener('click', e => {
      if (!menu.contains(e.target) && e.target !== trigger) closeMenu();
    });
    document.addEventListener('keydown', e => { if (e.key === 'Escape') closeMenu(); });
    menu.querySelectorAll('a').forEach(a => a.addEventListener('click', closeMenu));

    document.getElementById('share-other').addEventListener('click', async () => {
      const shareData = { title: document.title, url: location.href };
      if (navigator.share) {
        try { await navigator.share(shareData); } catch (_) { /* utente ha annullato */ }
      } else {
        try {
          await navigator.clipboard.writeText(location.href);
          toast.classList.add('show');
          setTimeout(() => toast.classList.remove('show'), 2000);
        } catch (_) { /* clipboard non disponibile, edge case trascurabile */ }
      }
      closeMenu();
    });
  })();
```

Sostituiscilo con (estrae `showToast()`, aggiunge il gestore per Instagram):

```js
  // ── Condivisione pagina ────────────────────────────────────────
  (() => {
    const trigger = document.getElementById('share-trigger');
    const menu    = document.getElementById('share-menu');
    const toast   = document.getElementById('share-toast');

    const pageUrl   = encodeURIComponent(location.href);
    const pageTitle = encodeURIComponent(document.title);
    document.getElementById('share-whatsapp').href = `https://wa.me/?text=${encodeURIComponent(document.title + ' - ' + location.href)}`;
    document.getElementById('share-facebook').href = `https://www.facebook.com/sharer/sharer.php?u=${pageUrl}`;
    document.getElementById('share-email').href    = `mailto:?subject=${pageTitle}&body=${pageUrl}`;

    const openMenu  = () => { menu.classList.add('show');    trigger.setAttribute('aria-expanded', 'true'); };
    const closeMenu = () => { menu.classList.remove('show'); trigger.setAttribute('aria-expanded', 'false'); };

    const showToast = text => {
      toast.textContent = text;
      toast.classList.add('show');
      setTimeout(() => toast.classList.remove('show'), 2000);
    };

    trigger.addEventListener('click', e => {
      e.stopPropagation();
      menu.classList.contains('show') ? closeMenu() : openMenu();
    });
    document.addEventListener('click', e => {
      if (!menu.contains(e.target) && e.target !== trigger) closeMenu();
    });
    document.addEventListener('keydown', e => { if (e.key === 'Escape') closeMenu(); });
    menu.querySelectorAll('a').forEach(a => a.addEventListener('click', closeMenu));

    document.getElementById('share-instagram').addEventListener('click', async () => {
      try {
        await navigator.clipboard.writeText(location.href);
        showToast('Link copiato! Incollalo su Instagram');
      } catch (_) { /* clipboard non disponibile, edge case trascurabile */ }
      closeMenu();
    });

    document.getElementById('share-other').addEventListener('click', async () => {
      const shareData = { title: document.title, url: location.href };
      if (navigator.share) {
        try { await navigator.share(shareData); } catch (_) { /* utente ha annullato */ }
      } else {
        try {
          await navigator.clipboard.writeText(location.href);
          showToast('Link copiato!');
        } catch (_) { /* clipboard non disponibile, edge case trascurabile */ }
      }
      closeMenu();
    });
  })();
```

- [ ] **Step 4: Applica lo stesso identico JS in `en/index.html`, con questa unica differenza testuale**

Stesso punto, stesso codice, tranne le due stringhe passate a `showToast`:
- `showToast('Link copiato! Incollalo su Instagram')` → `showToast('Link copied! Paste it on Instagram')`
- `showToast('Link copiato!')` → `showToast('Link copied!')`

- [ ] **Step 5: Verifica manuale**

```bash
open /Users/wepan/repo/officina-camper/index.html
```
1. Apri il menu condivisione → **atteso:** ora appare anche "Instagram" tra WhatsApp e Facebook, con un'icona a fotocamera stilizzata.
2. Clicca "Instagram" → **atteso:** il menu si chiude, e per ~2 secondi appare il fumetto "Link copiato! Incollalo su Instagram" (su Chrome/Safari desktop il link viene effettivamente copiato negli appunti).
3. Clicca di nuovo "Altro" → **atteso:** il fumetto mostra "Link copiato!" (senza il riferimento a Instagram) — conferma che `showToast` cambia correttamente testo in base al pulsante cliccato.

Ripeti su `en/index.html`: fumetto "Link copied! Paste it on Instagram" per Instagram, "Link copied!" per "More".

- [ ] **Step 6: Verifica screenshot mobile (nessun overflow con la voce in più)**

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless --disable-gpu --hide-scrollbars --window-size=500,900 --screenshot=/tmp/task4-it.png "file:///Users/wepan/repo/officina-camper/index.html"
```
Expected: il menu (visibile solo se aperto via interazione, quindi in questo screenshot statico resta chiuso) e l'header non presentano regressioni — la voce in più sta nel menu a comparsa, non nella barra dell'header, quindi non incide sulla larghezza dell'header.

- [ ] **Step 7: Commit e push**

```bash
git add index.html en/index.html
git commit -m "Aggiunge Instagram come voce dedicata nel menu di condivisione"
git push
```
