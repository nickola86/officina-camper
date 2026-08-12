# Popup chiusura per ferie di agosto Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Mostrare, poco dopo il caricamento della home page (IT e EN), un popup modale che informa gli utenti della chiusura per ferie di agosto e del supporto solo telefonico, senza che il testo sia presente nel sorgente HTML statico.

**Architecture:** Sito statico senza build/test tooling (solo HTML + Tailwind CDN + CSS custom in `<style>` + un unico `<script>` vanilla JS a fondo pagina), duplicato in `index.html` (IT) e `en/index.html` (EN). Il popup viene aggiunto come: (1) regole CSS nel blocco `<style>` esistente (nessun testo, solo classi/animazioni), (2) un IIFE JS in coda allo script esistente che, dopo `window.addEventListener('load', …)` e un `setTimeout` di 1s, costruisce l'overlay/card via `document.createElement`/`textContent` e li appende al `<body>` — quindi il testo dell'avviso non esiste mai nel sorgente HTML scaricato.

**Tech Stack:** HTML statico, Tailwind CDN, CSS vanilla, JavaScript vanilla (nessun framework, nessun bundler, nessun test runner).

## Global Constraints

- Nessun markup del popup deve comparire nell'HTML statico: solo CSS (senza testo) e JS che costruisce il DOM a runtime.
- Popup mostrato ~1000ms dopo l'evento `load`.
- Persistenza chiusura: `localStorage` con chiave `oc-august-notice-dismissed`, condivisa tra IT ed EN (stesso dominio).
- Chiudibile via bottone "×", click sull'overlay fuori dalla card, tasto `Escape`.
- CTA telefonica: `tel:+393513531335` con `data-cta="tel_august_popup"` (per intercettazione automatica dal tracking GA esistente — nessun codice di tracking aggiuntivo).
- Palette da riusare: `--c-cream`, `--c-amber`, `--c-forest-dk`, `--c-text`, `--c-muted` (già definite in entrambi i file).
- Testi IT: titolo "Chiusi per ferie"; corpo "Saremo chiusi dal 1° al 31 agosto. In questo periodo possiamo garantire solo supporto telefonico."; CTA "📞 Chiama ora"; aria-label card "Avviso chiusura per ferie".
- Testi EN: titolo "Closed for summer break"; corpo "We're closed from August 1 to 31. During this period we can only offer phone support."; CTA "📞 Call now"; aria-label card "Summer closure notice".
- Nessun test runner nel progetto: la verifica di ogni task è manuale, tramite un piccolo server statico locale e ispezione nel browser (DOM, console, `localStorage`).

---

### Task 1: Popup chiusura agosto — `index.html` (IT)

**Files:**
- Modify: `index.html:585-587` (blocco `<style>`, aggiunta regole CSS prima di `</style>`)
- Modify: `index.html:1198-1200` (blocco `<script>`, aggiunta IIFE prima di `</script>`)

**Interfaces:**
- Consumes: variabili CSS già definite in `index.html` (`--c-cream`, `--c-amber`, `--c-forest-dk`, `--c-text`, `--c-muted`); pattern di tracking GA esistente che ascolta `[data-cta]` su `document`.
- Produces: classi CSS `.oc-modal-overlay`, `.oc-modal-card`, `.oc-modal-close`, `.oc-modal-cta`; funzione JS `showAugustNotice()` (interna all'IIFE, non esposta globalmente); chiave `localStorage` `oc-august-notice-dismissed` (riusata identica in Task 2).

- [ ] **Step 1: Aggiungere le regole CSS del popup**

In `index.html`, trova il blocco:

```html
    .btn-cookie-solid   { background: var(--c-amber); color: #fff; }
    .btn-cookie-outline { background: transparent; color: var(--c-cream); border: 1.5px solid rgba(255,255,255,.3); }
    @media (min-width: 640px) {
      #cookie-banner { flex-direction: row; align-items: center; justify-content: space-between; }
    }

  </style>
```

Sostituiscilo con (aggiunta delle regole `.oc-modal-*` subito prima di `</style>`):

```html
    .btn-cookie-solid   { background: var(--c-amber); color: #fff; }
    .btn-cookie-outline { background: transparent; color: var(--c-cream); border: 1.5px solid rgba(255,255,255,.3); }
    @media (min-width: 640px) {
      #cookie-banner { flex-direction: row; align-items: center; justify-content: space-between; }
    }

    /* ── Popup chiusura agosto ─────────────────────────────── */
    .oc-modal-overlay {
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,.55);
      z-index: 10010;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 1.25rem;
      opacity: 0;
      animation: oc-fade-in .25s ease forwards;
    }
    .oc-modal-card {
      position: relative;
      background: var(--c-cream);
      border-top: 4px solid var(--c-amber);
      border-radius: 14px;
      max-width: 380px;
      width: 100%;
      padding: 2rem 1.75rem 1.75rem;
      box-shadow: 0 20px 60px rgba(0,0,0,.35);
      text-align: center;
      transform: scale(.95);
      animation: oc-scale-in .25s ease forwards;
    }
    .oc-modal-card h2 {
      margin: 0 0 .6rem;
      font-size: 1.25rem;
      font-weight: 800;
      color: var(--c-forest-dk);
    }
    .oc-modal-card p {
      margin: 0 0 1.25rem;
      font-size: .92rem;
      line-height: 1.6;
      color: var(--c-text);
    }
    .oc-modal-close {
      position: absolute;
      top: .6rem;
      right: .75rem;
      background: none;
      border: none;
      font-size: 1.3rem;
      line-height: 1;
      color: var(--c-muted);
      cursor: pointer;
      padding: .25rem;
    }
    .oc-modal-cta {
      display: inline-block;
      background: var(--c-amber);
      color: #fff;
      font-weight: 700;
      font-size: .95rem;
      padding: .7rem 1.4rem;
      border-radius: 10px;
      text-decoration: none;
    }
    @keyframes oc-fade-in { to { opacity: 1; } }
    @keyframes oc-scale-in { to { transform: scale(1); } }

  </style>
```

- [ ] **Step 2: Aggiungere l'IIFE JS che genera il popup**

In `index.html`, trova il blocco finale dello script:

```html
  // ── Tracciamento click sulle CTA (WhatsApp, telefono, Maps) ───
  document.addEventListener('click', e => {
    const cta = e.target.closest('[data-cta]');
    if (!cta || typeof gtag !== 'function') return;
    gtag('event', 'cta_click', {
      cta_id: cta.dataset.cta,
      cta_text: cta.textContent.trim().replace(/\s+/g, ' '),
      cta_url: cta.href,
      page_location: window.location.href
    });
  });
</script>
```

Sostituiscilo con (aggiunta dell'IIFE `oc-modal` subito prima di `</script>`):

```html
  // ── Tracciamento click sulle CTA (WhatsApp, telefono, Maps) ───
  document.addEventListener('click', e => {
    const cta = e.target.closest('[data-cta]');
    if (!cta || typeof gtag !== 'function') return;
    gtag('event', 'cta_click', {
      cta_id: cta.dataset.cta,
      cta_text: cta.textContent.trim().replace(/\s+/g, ' '),
      cta_url: cta.href,
      page_location: window.location.href
    });
  });

  // ── Popup chiusura agosto ────────────────────────────────────
  (function () {
    const DISMISS_KEY = 'oc-august-notice-dismissed';

    function showAugustNotice() {
      if (localStorage.getItem(DISMISS_KEY)) return;

      const overlay = document.createElement('div');
      overlay.className = 'oc-modal-overlay';
      overlay.setAttribute('role', 'presentation');

      const card = document.createElement('div');
      card.className = 'oc-modal-card';
      card.setAttribute('role', 'dialog');
      card.setAttribute('aria-modal', 'true');
      card.setAttribute('aria-label', 'Avviso chiusura per ferie');

      const closeBtn = document.createElement('button');
      closeBtn.className = 'oc-modal-close';
      closeBtn.setAttribute('aria-label', 'Chiudi');
      closeBtn.textContent = '×';

      const title = document.createElement('h2');
      title.textContent = 'Chiusi per ferie';

      const body = document.createElement('p');
      body.textContent = 'Saremo chiusi dal 1° al 31 agosto. In questo periodo possiamo garantire solo supporto telefonico.';

      const cta = document.createElement('a');
      cta.className = 'oc-modal-cta';
      cta.href = 'tel:+393513531335';
      cta.setAttribute('data-cta', 'tel_august_popup');
      cta.textContent = '📞 Chiama ora';

      card.appendChild(closeBtn);
      card.appendChild(title);
      card.appendChild(body);
      card.appendChild(cta);
      overlay.appendChild(card);
      document.body.appendChild(overlay);

      function dismiss() {
        localStorage.setItem(DISMISS_KEY, '1');
        overlay.remove();
        document.removeEventListener('keydown', onKeydown);
      }

      function onKeydown(e) {
        if (e.key === 'Escape') dismiss();
      }

      closeBtn.addEventListener('click', dismiss);
      overlay.addEventListener('click', e => { if (e.target === overlay) dismiss(); });
      document.addEventListener('keydown', onKeydown);

      closeBtn.focus();
    }

    window.addEventListener('load', () => { setTimeout(showAugustNotice, 1000); });
  })();
</script>
```

- [ ] **Step 3: Verifica manuale nel browser**

Avvia un server statico locale e apri la home IT:

```bash
cd /Users/wepan/repo/officina-camper && python3 -m http.server 8000
```

Apri `http://localhost:8000/` nel browser:
1. Attendi ~1s dopo il caricamento: il popup deve apparire centrato con overlay scuro dietro.
2. Verifica che il testo sia "Chiusi per ferie" / "Saremo chiusi dal 1° al 31 agosto..." e che il bottone "📞 Chiama ora" abbia `href="tel:+393513531335"`.
3. Chiudi col bottone "×": il popup scompare. Ricarica la pagina: il popup NON deve ripresentarsi (persistenza `localStorage`).
4. In DevTools → Application → Local Storage, verifica la presenza della chiave `oc-august-notice-dismissed` con valore `1`.
5. Rimuovi la chiave da `localStorage`, ricarica: il popup deve ricomparire. Chiudilo stavolta cliccando sull'overlay fuori dalla card: deve chiudersi e la chiave deve ripresentarsi in `localStorage`.
6. Rimuovi di nuovo la chiave, ricarica, attendi il popup, premi `Escape`: deve chiudersi.
7. In DevTools → Elements, verifica che al primo caricamento (prima del `setTimeout`) il DOM iniziale (view-source) non contenga il testo del popup — usa "View Page Source" (Ctrl/Cmd+U): il testo "Chiusi per ferie" NON deve comparire nel sorgente statico.
8. Ferma il server (`Ctrl+C`).

Expected: tutti i controlli sopra passano senza errori in console.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "$(cat <<'EOF'
Aggiunge popup di chiusura per ferie di agosto (IT)

Generato interamente via JS dopo il load per non essere letto dai
crawler; persiste la chiusura in localStorage.
EOF
)"
```

---

### Task 2: Popup chiusura agosto — `en/index.html` (EN)

**Files:**
- Modify: `en/index.html:584-587` (blocco `<style>`, aggiunta regole CSS prima di `</style>`)
- Modify: `en/index.html:1197-1200` (blocco `<script>`, aggiunta IIFE prima di `</script>`)

**Interfaces:**
- Consumes: stesse variabili CSS di Task 1 (già presenti in `en/index.html`, identico stylesheet); stesso pattern di tracking GA.
- Produces: stesse classi CSS di Task 1 (`.oc-modal-*`); stessa chiave `localStorage` `oc-august-notice-dismissed` (deve restare identica a Task 1 — condivisa tra le due pagine sullo stesso dominio).

- [ ] **Step 1: Aggiungere le regole CSS del popup (identiche a Task 1)**

In `en/index.html`, trova lo stesso blocco di chiusura dello `<style>`:

```html
    .btn-cookie-solid   { background: var(--c-amber); color: #fff; }
    .btn-cookie-outline { background: transparent; color: var(--c-cream); border: 1.5px solid rgba(255,255,255,.3); }
    @media (min-width: 640px) {
      #cookie-banner { flex-direction: row; align-items: center; justify-content: space-between; }
    }

  </style>
```

Sostituiscilo con lo stesso identico blocco `.oc-modal-*` usato in Task 1 Step 1:

```html
    .btn-cookie-solid   { background: var(--c-amber); color: #fff; }
    .btn-cookie-outline { background: transparent; color: var(--c-cream); border: 1.5px solid rgba(255,255,255,.3); }
    @media (min-width: 640px) {
      #cookie-banner { flex-direction: row; align-items: center; justify-content: space-between; }
    }

    /* ── Popup chiusura agosto ─────────────────────────────── */
    .oc-modal-overlay {
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,.55);
      z-index: 10010;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 1.25rem;
      opacity: 0;
      animation: oc-fade-in .25s ease forwards;
    }
    .oc-modal-card {
      position: relative;
      background: var(--c-cream);
      border-top: 4px solid var(--c-amber);
      border-radius: 14px;
      max-width: 380px;
      width: 100%;
      padding: 2rem 1.75rem 1.75rem;
      box-shadow: 0 20px 60px rgba(0,0,0,.35);
      text-align: center;
      transform: scale(.95);
      animation: oc-scale-in .25s ease forwards;
    }
    .oc-modal-card h2 {
      margin: 0 0 .6rem;
      font-size: 1.25rem;
      font-weight: 800;
      color: var(--c-forest-dk);
    }
    .oc-modal-card p {
      margin: 0 0 1.25rem;
      font-size: .92rem;
      line-height: 1.6;
      color: var(--c-text);
    }
    .oc-modal-close {
      position: absolute;
      top: .6rem;
      right: .75rem;
      background: none;
      border: none;
      font-size: 1.3rem;
      line-height: 1;
      color: var(--c-muted);
      cursor: pointer;
      padding: .25rem;
    }
    .oc-modal-cta {
      display: inline-block;
      background: var(--c-amber);
      color: #fff;
      font-weight: 700;
      font-size: .95rem;
      padding: .7rem 1.4rem;
      border-radius: 10px;
      text-decoration: none;
    }
    @keyframes oc-fade-in { to { opacity: 1; } }
    @keyframes oc-scale-in { to { transform: scale(1); } }

  </style>
```

- [ ] **Step 2: Aggiungere l'IIFE JS con copy inglese**

In `en/index.html`, trova lo stesso blocco finale dello script (identico a Task 1):

```html
  // ── Tracciamento click sulle CTA (WhatsApp, telefono, Maps) ───
  document.addEventListener('click', e => {
    const cta = e.target.closest('[data-cta]');
    if (!cta || typeof gtag !== 'function') return;
    gtag('event', 'cta_click', {
      cta_id: cta.dataset.cta,
      cta_text: cta.textContent.trim().replace(/\s+/g, ' '),
      cta_url: cta.href,
      page_location: window.location.href
    });
  });
</script>
```

Sostituiscilo con (testi in inglese, stessa `DISMISS_KEY` e stesso `data-cta`):

```html
  // ── Tracciamento click sulle CTA (WhatsApp, telefono, Maps) ───
  document.addEventListener('click', e => {
    const cta = e.target.closest('[data-cta]');
    if (!cta || typeof gtag !== 'function') return;
    gtag('event', 'cta_click', {
      cta_id: cta.dataset.cta,
      cta_text: cta.textContent.trim().replace(/\s+/g, ' '),
      cta_url: cta.href,
      page_location: window.location.href
    });
  });

  // ── August closure popup ─────────────────────────────────────
  (function () {
    const DISMISS_KEY = 'oc-august-notice-dismissed';

    function showAugustNotice() {
      if (localStorage.getItem(DISMISS_KEY)) return;

      const overlay = document.createElement('div');
      overlay.className = 'oc-modal-overlay';
      overlay.setAttribute('role', 'presentation');

      const card = document.createElement('div');
      card.className = 'oc-modal-card';
      card.setAttribute('role', 'dialog');
      card.setAttribute('aria-modal', 'true');
      card.setAttribute('aria-label', 'Summer closure notice');

      const closeBtn = document.createElement('button');
      closeBtn.className = 'oc-modal-close';
      closeBtn.setAttribute('aria-label', 'Close');
      closeBtn.textContent = '×';

      const title = document.createElement('h2');
      title.textContent = 'Closed for summer break';

      const body = document.createElement('p');
      body.textContent = 'We’re closed from August 1 to 31. During this period we can only offer phone support.';

      const cta = document.createElement('a');
      cta.className = 'oc-modal-cta';
      cta.href = 'tel:+393513531335';
      cta.setAttribute('data-cta', 'tel_august_popup');
      cta.textContent = '📞 Call now';

      card.appendChild(closeBtn);
      card.appendChild(title);
      card.appendChild(body);
      card.appendChild(cta);
      overlay.appendChild(card);
      document.body.appendChild(overlay);

      function dismiss() {
        localStorage.setItem(DISMISS_KEY, '1');
        overlay.remove();
        document.removeEventListener('keydown', onKeydown);
      }

      function onKeydown(e) {
        if (e.key === 'Escape') dismiss();
      }

      closeBtn.addEventListener('click', dismiss);
      overlay.addEventListener('click', e => { if (e.target === overlay) dismiss(); });
      document.addEventListener('keydown', onKeydown);

      closeBtn.focus();
    }

    window.addEventListener('load', () => { setTimeout(showAugustNotice, 1000); });
  })();
</script>
```

- [ ] **Step 3: Verifica manuale nel browser**

Con il server statico ancora attivo (o riavvialo: `python3 -m http.server 8000` dalla root del repo), apri `http://localhost:8000/en/`:

1. Se la chiave `oc-august-notice-dismissed` è ancora impostata da Task 1 (stesso `localhost:8000` = stessa origin), rimuovila da DevTools → Application → Local Storage prima di procedere.
2. Attendi ~1s: il popup deve apparire con testo inglese "Closed for summer break" / "We're closed from August 1 to 31..." e bottone "📞 Call now" con `href="tel:+393513531335"`.
3. Chiudi col bottone "×", ricarica: non deve ripresentarsi.
4. Verifica cross-lingua: dopo aver chiuso il popup su `/en/`, apri `http://localhost:8000/` (IT) — il popup NON deve apparire (stessa chiave `localStorage`, stessa origin).
5. Rimuovi la chiave, ricarica `/en/`, verifica chiusura via click sull'overlay e via `Escape` (come Task 1 Step 3).
6. View Page Source su `/en/`: il testo "Closed for summer break" NON deve comparire nel sorgente statico.
7. Ferma il server.

Expected: tutti i controlli sopra passano senza errori in console.

- [ ] **Step 4: Commit**

```bash
git add en/index.html
git commit -m "$(cat <<'EOF'
Aggiunge popup di chiusura per ferie di agosto (EN)

Stessa logica della versione IT, testi tradotti, chiave localStorage
condivisa tra le due pagine.
EOF
)"
```

---

### Task 3: Push

**Files:** nessuna modifica — solo push dei commit di Task 1 e Task 2.

- [ ] **Step 1: Verificare lo stato del branch**

```bash
git status
git log --oneline -5
```

Expected: working tree pulito, gli ultimi commit sono quelli di Task 1 e Task 2 (oltre al commit dello spec doc).

- [ ] **Step 2: Push su remote**

```bash
git push
```

Expected: push completato senza conflitti sul branch corrente (`main`).
