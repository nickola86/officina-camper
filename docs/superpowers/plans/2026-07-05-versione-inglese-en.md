# Versione Inglese del Sito (IT/EN) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Aggiungere una versione inglese del sito (`en/index.html`) affiancata all'italiana (`index.html`), con SEO multilingua corretto (hreflang), uno switcher di lingua in navbar e un banner di suggerimento basato sulla lingua del browser, senza redirect automatici.

**Architecture:** Due file HTML statici indipendenti che condividono la stessa struttura/CSS/JS (nessuna build pipeline). `index.html` resta la versione italiana canonical alla root; `en/index.html` è la copia tradotta in una sottocartella. Ogni pagina dichiara `hreflang` reciproci verso l'altra.

**Tech Stack:** HTML/CSS/JS vanilla, Tailwind CDN (invariato), nessun bundler, nessun framework.

## Global Constraints

- Nessuna build pipeline: i due file HTML vengono editati e mantenuti a mano, non generati.
- `GA_MEASUREMENT_ID` invariato su entrambe le pagine: `G-W8RP91ZB7B`.
- Link Google Maps e query dell'indirizzo reale (Villamassargia) invariati in entrambe le lingue: `https://maps.google.com/?q=Az.+Agricola+Vacca+Monte+Cadelano+Villamassargia+SU`.
- Numero WhatsApp/telefono invariato: `wa.me/393513531335`, `tel:+393513531335`. Solo il testo visualizzato del telefono cambia in EN (`+39 351 353 1335`).
- Placeholder JSON-LD ancora da compilare (`streetAddress`, `postalCode`, `openingHours`) restano placeholder, tradotti in inglese, senza inventare dati reali.
- Nessun redirect automatico basato su lingua/geolocalizzazione: solo banner dismissabile + switcher manuale.
- Toponimi (Iglesias, Carbonia, Domusnovas, Gonnesa, Musei, "Iglesias Camper Service") invariati in entrambe le lingue.
- Asset condivisi (`assets/hero-van.png`, `favicon.svg`) referenziati con path assoluti (`/assets/...`, `/favicon.svg`) in `en/index.html`.

---

### Task 1: Infrastruttura lingua su `index.html` (IT) — hreflang, switcher, banner

**Files:**
- Modify: `index.html`

**Interfaces:**
- Produces: markup/CSS/JS che `en/index.html` copierà come base nel Task 2 (id `#lang-banner`, `#lang-banner-close`, classe `.lang-banner-switch`, `#lang-switch-nav`, variabili JS `LANG_CURRENT`/`LANG_ALT`/`LANG_ALT_URL`).

- [ ] **Step 1: Aggiungere i tag hreflang/og:locale nel `<head>`**

In `index.html`, subito dopo la riga `<meta property="og:url" content="https://iglesiascamperservice.com/">` (riga 15), inserire:

```html
  <link rel="alternate" hreflang="it" href="https://iglesiascamperservice.com/">
  <link rel="alternate" hreflang="en" href="https://iglesiascamperservice.com/en/">
  <link rel="alternate" hreflang="x-default" href="https://iglesiascamperservice.com/">
  <meta property="og:locale" content="it_IT">
  <meta property="og:locale:alternate" content="en_US">
```

- [ ] **Step 2: Verificare che i tag siano presenti**

Run: `grep -n "hreflang\|og:locale" index.html`
Expected output: 5 righe corrispondenti ai tag appena aggiunti.

- [ ] **Step 3: Aggiungere CSS per banner lingua e switcher navbar**

Nel blocco `<style>`, subito dopo la regola `.divider { ... }` (dopo la riga con `opacity: .18;` e la chiusura `}` del blocco `.divider`), aggiungere:

```css
    /* ── Language banner ───────────────────────────────────── */
    #lang-banner {
      display: none;
      background: var(--c-cream-warm);
      border-bottom: 1px solid var(--c-border);
    }
    #lang-banner.show { display: block; }
    .lang-banner-inner {
      max-width: 1100px;
      margin: 0 auto;
      padding: .75rem 1.5rem;
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 1rem;
      flex-wrap: wrap;
    }
    .lang-banner-inner p { margin: 0; font-size: .85rem; color: var(--c-text); }
    .lang-banner-actions { display: flex; align-items: center; gap: .75rem; }
    .lang-banner-switch {
      background: var(--c-amber);
      color: #fff;
      font-weight: 700;
      font-size: .82rem;
      padding: .45rem .9rem;
      border-radius: 8px;
      text-decoration: none;
      white-space: nowrap;
    }
    #lang-banner-close {
      background: none;
      border: none;
      font-size: 1.1rem;
      line-height: 1;
      color: var(--c-muted);
      cursor: pointer;
      padding: .25rem;
    }

    /* ── Language switcher (navbar) ────────────────────────── */
    .lang-switch {
      color: var(--c-forest-lt);
      font-weight: 700;
      font-size: .8rem;
      text-decoration: none;
      padding: .3rem .5rem;
      flex-shrink: 0;
    }
    .lang-switch:hover { color: var(--c-cream); }
```

- [ ] **Step 4: Verificare che il CSS sia stato inserito**

Run: `grep -n "lang-banner\|lang-switch" index.html | head -5`
Expected: righe che mostrano le nuove regole CSS.

- [ ] **Step 5: Aggiungere il link switcher "EN" in navbar**

In `index.html`, nel blocco navbar, subito prima del link CTA WhatsApp (`<a href="https://wa.me/393513531335" data-cta="whatsapp_header"`, riga 504), inserire:

```html
      <!-- Language switcher -->
      <a href="/en/" id="lang-switch-nav" class="lang-switch" aria-label="Switch to English">EN</a>

```

- [ ] **Step 6: Aggiungere il markup del banner lingua**

Subito dopo la chiusura del tag `</header>` (riga 515) e prima del commento `<!-- ════════════════════════════════════════ HERO ═══════════ -->`, inserire:

```html
  <!-- ══════════════════════ LANGUAGE BANNER ═══════════════════ -->
  <div id="lang-banner" role="region" aria-label="Suggerimento lingua">
    <div class="lang-banner-inner">
      <p>Would you prefer English?</p>
      <div class="lang-banner-actions">
        <a href="/en/" data-cta="lang_switch_banner" class="lang-banner-switch">Switch</a>
        <button id="lang-banner-close" aria-label="Chiudi">×</button>
      </div>
    </div>
  </div>

```

- [ ] **Step 7: Verificare la struttura HTML aggiunta**

Run: `grep -n "lang-switch-nav\|id=\"lang-banner\"" index.html`
Expected: 2 righe che mostrano il link switcher e il div banner.

- [ ] **Step 8: Aggiungere la logica JS del banner/switcher**

Nel blocco `<script>` finale, subito prima della sezione `// ── Google Analytics (GA4), caricato solo dopo consenso ───────` (riga 943), inserire:

```javascript
  // ── Language banner & switcher ───────────────────────────────
  const LANG_CURRENT = 'it';
  const LANG_ALT = 'en';
  (function () {
    const banner = document.getElementById('lang-banner');
    const pref = localStorage.getItem('oc-lang-pref');
    const dismissed = localStorage.getItem('oc-lang-banner-dismissed');
    const browserLang = (navigator.language || '').slice(0, 2).toLowerCase();
    if (!pref && !dismissed && browserLang === LANG_ALT) {
      banner.classList.add('show');
    }
    document.getElementById('lang-banner-close').addEventListener('click', () => {
      localStorage.setItem('oc-lang-banner-dismissed', '1');
      banner.classList.remove('show');
    });
    document.querySelector('.lang-banner-switch').addEventListener('click', () => {
      localStorage.setItem('oc-lang-pref', LANG_ALT);
    });
    document.getElementById('lang-switch-nav').addEventListener('click', () => {
      localStorage.setItem('oc-lang-pref', LANG_ALT);
    });
  })();
```

- [ ] **Step 9: Verificare che `index.html` sia sintatticamente valido (nessun tag rotto)**

Run: `node -e "require('fs').readFileSync('index.html','utf8')" && echo OK`
Expected: `OK` (verifica solo che il file sia leggibile; l'HTML non è XML quindi non c'è un validatore rigoroso — la verifica visiva in Task 8 copre la correttezza reale).

- [ ] **Step 10: Commit**

```bash
git add index.html
git commit -m "Aggiunge hreflang, switcher lingua e banner suggerimento EN a index.html"
```

---

### Task 2: Creare `en/index.html` — copia base + traduzione `<head>`

**Files:**
- Create: `en/index.html` (copiato da `index.html` dopo il Task 1)

**Interfaces:**
- Consumes: `index.html` aggiornato dal Task 1 (contiene già markup/CSS/JS di banner e switcher, da tradurre qui).
- Produces: `en/index.html` con `<head>` completo in inglese; le sezioni body vengono tradotte nei Task 3-6.

- [ ] **Step 1: Creare la cartella e copiare il file**

```bash
mkdir -p en
cp index.html en/index.html
```

- [ ] **Step 2: Tradurre `lang`, title, meta description**

In `en/index.html`:

old:
```html
<html lang="it" class="scroll-smooth">
```
new:
```html
<html lang="en" class="scroll-smooth">
```

old:
```html
  <meta name="description" content="Iglesias Camper Service: officina specializzata in camper e van nel Sulcis Iglesiente. Impianti di bordo, camperizzazioni su misura, manutenzione e sanificazione. Vicino alla SS130, Iglesias (SU).">
```
new:
```html
  <meta name="description" content="Iglesias Camper Service: specialist workshop for campers and vans in Sulcis Iglesiente, Sardinia. Onboard systems, custom camper conversions, maintenance and sanitization. Near the SS130 highway, Iglesias (SU), Italy.">
```

old:
```html
  <title>Iglesias Camper Service — Assistenza camper professionale</title>
```
new:
```html
  <title>Iglesias Camper Service — Professional Camper Service in Sardinia</title>
```

- [ ] **Step 3: Tradurre canonical, hreflang, og:*, e correggere gli asset**

old:
```html
  <link rel="canonical" href="https://iglesiascamperservice.com/">
  <meta property="og:title" content="Iglesias Camper Service">
  <meta property="og:description" content="Assistenza tecnica, impianti di bordo e camperizzazioni su misura. Iglesias, Sulcis Iglesiente.">
  <meta property="og:type" content="website">
  <meta property="og:url" content="https://iglesiascamperservice.com/">
  <link rel="alternate" hreflang="it" href="https://iglesiascamperservice.com/">
  <link rel="alternate" hreflang="en" href="https://iglesiascamperservice.com/en/">
  <link rel="alternate" hreflang="x-default" href="https://iglesiascamperservice.com/">
  <meta property="og:locale" content="it_IT">
  <meta property="og:locale:alternate" content="en_US">
```
new:
```html
  <link rel="canonical" href="https://iglesiascamperservice.com/en/">
  <meta property="og:title" content="Iglesias Camper Service">
  <meta property="og:description" content="Technical service, onboard systems and custom camper conversions. Iglesias, Sulcis Iglesiente, Sardinia.">
  <meta property="og:type" content="website">
  <meta property="og:url" content="https://iglesiascamperservice.com/en/">
  <link rel="alternate" hreflang="it" href="https://iglesiascamperservice.com/">
  <link rel="alternate" hreflang="en" href="https://iglesiascamperservice.com/en/">
  <link rel="alternate" hreflang="x-default" href="https://iglesiascamperservice.com/">
  <meta property="og:locale" content="en_US">
  <meta property="og:locale:alternate" content="it_IT">
```

Anche il favicon deve usare un path assoluto (già lo è: `href="/favicon.svg"` — verificare che non cambi).

- [ ] **Step 4: Tradurre il JSON-LD `AutoRepair`**

old:
```html
    "description": "Officina specializzata in camper e van a Iglesias, Sulcis Iglesiente: impianti di bordo, igiene e sanificazione, falegnameria e modifiche, manutenzione e riparazioni, camperizzazioni su misura.",
    "url": "https://iglesiascamperservice.com/",
```
new:
```html
    "description": "Specialist workshop for campers and vans in Iglesias, Sulcis Iglesiente (Sardinia, Italy): onboard systems, hygiene and sanitization, carpentry and modifications, maintenance and repairs, custom camper conversions.",
    "url": "https://iglesiascamperservice.com/en/",
```

old:
```html
      "streetAddress": "[INSERIRE INDIRIZZO PRECISO]",
      "addressLocality": "Iglesias",
      "postalCode": "[INSERIRE CAP]",
```
new:
```html
      "streetAddress": "[ENTER STREET ADDRESS]",
      "addressLocality": "Iglesias",
      "postalCode": "[ENTER POSTAL CODE]",
```

old:
```html
    "openingHours": "[INSERIRE ORARI, es. Mo-Fr 08:30-18:00]",
```
new:
```html
    "openingHours": "[ENTER HOURS, e.g. Mon-Fri 08:30-18:00]",
```

`areaServed` (Iglesias, Carbonia, Domusnovas, Gonnesa, Musei) e `hasMap` restano invariati.

- [ ] **Step 5: Tradurre il JSON-LD `FAQPage`**

Sostituire l'intero blocco `"mainEntity": [ ... ]` con:

```json
    "mainEntity": [
      {
        "@type": "Question",
        "name": "Which areas do you serve?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "Our workshop is located in Iglesias, a few minutes from the SS130 highway. We serve the whole Sulcis Iglesiente area: Iglesias, Carbonia, Domusnovas, Gonnesa and Musei."
        }
      },
      {
        "@type": "Question",
        "name": "Do you also do full van conversions?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "Yes. We design and build complete conversions of vans and special vehicles, from concept to delivery, based on the needs of those who travel."
        }
      },
      {
        "@type": "Question",
        "name": "Do I need an appointment, or can I just stop by?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "It's best to contact us first on WhatsApp or by phone at +39 351 353 1335: that way we can discuss the job together and agree on a day and time, with no waiting around."
        }
      },
      {
        "@type": "Question",
        "name": "Do you clean and sanitize water tanks?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "Yes. We sanitize fresh, grey and black water tanks and disinfect camper interiors, including upholstery and carpets."
        }
      },
      {
        "@type": "Question",
        "name": "How does the quote work?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "The quote is always agreed before starting any work, with no surprises on the invoice. Project assessment is free of charge, with no obligation."
        }
      },
      {
        "@type": "Question",
        "name": "Do you only work on campers, or vans too?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "We work on campers, vans and camperized vans: technical service, onboard systems, maintenance and custom fit-outs."
        }
      }
    ]
```

- [ ] **Step 6: Validare la sintassi dei due blocchi JSON-LD**

Run:
```bash
node -e "
const fs = require('fs');
const html = fs.readFileSync('en/index.html', 'utf8');
const blocks = [...html.matchAll(/<script type=\"application\/ld\+json\">([\s\S]*?)<\/script>/g)];
blocks.forEach((b, i) => { JSON.parse(b[1]); console.log('JSON-LD block', i, 'OK'); });
"
```
Expected: `JSON-LD block 0 OK` e `JSON-LD block 1 OK`.

- [ ] **Step 7: Commit**

```bash
git add en/index.html
git commit -m "Crea en/index.html: copia base e traduzione head/JSON-LD"
```

---

### Task 3: Tradurre navbar, switcher, banner lingua, hero, strip posizione in `en/index.html`

**Files:**
- Modify: `en/index.html`

- [ ] **Step 1: Tradurre navbar (nav-link, CTA header) e switcher lingua**

old:
```html
      <!-- Nav links (desktop only) -->
      <nav class="hidden md:flex items-center gap-8" aria-label="Navigazione principale">
        <a href="#chi-siamo" class="nav-link">Chi siamo</a>
        <a href="#servizi"   class="nav-link">Servizi</a>
        <a href="#contatti"  class="nav-link">Dove siamo</a>
        <a href="#faq"       class="nav-link">FAQ</a>
      </nav>

      <!-- Language switcher -->
      <a href="/en/" id="lang-switch-nav" class="lang-switch" aria-label="Switch to English">EN</a>

      <!-- CTA -->
      <a href="https://wa.me/393513531335" data-cta="whatsapp_header"
         class="flex items-center gap-1.5 text-white font-bold rounded-full px-4 py-2 no-underline flex-shrink-0"
         style="background:var(--c-wa);font-size:.75rem"
         aria-label="Contattaci su WhatsApp">
```
new:
```html
      <!-- Nav links (desktop only) -->
      <nav class="hidden md:flex items-center gap-8" aria-label="Main navigation">
        <a href="#chi-siamo" class="nav-link">About us</a>
        <a href="#servizi"   class="nav-link">Services</a>
        <a href="#contatti"  class="nav-link">Location</a>
        <a href="#faq"       class="nav-link">FAQ</a>
      </nav>

      <!-- Language switcher -->
      <a href="/" id="lang-switch-nav" class="lang-switch" aria-label="Passa all'italiano">IT</a>

      <!-- CTA -->
      <a href="https://wa.me/393513531335" data-cta="whatsapp_header"
         class="flex items-center gap-1.5 text-white font-bold rounded-full px-4 py-2 no-underline flex-shrink-0"
         style="background:var(--c-wa);font-size:.75rem"
         aria-label="Contact us on WhatsApp">
```

- [ ] **Step 2: Tradurre il testo "Contattaci" del CTA header e l'aria-label del logo**

old:
```html
      <a href="#top" class="flex items-center gap-2 no-underline flex-shrink-0" aria-label="Torna in cima">
```
new:
```html
      <a href="#top" class="flex items-center gap-2 no-underline flex-shrink-0" aria-label="Back to top">
```

old (subito dopo l'SVG WhatsApp nel CTA header):
```html
        Contattaci
      </a>
    </div>
  </header>
```
new:
```html
        Contact us
      </a>
    </div>
  </header>
```

- [ ] **Step 3: Tradurre il banner lingua e invertire la config JS**

old (markup):
```html
  <!-- ══════════════════════ LANGUAGE BANNER ═══════════════════ -->
  <div id="lang-banner" role="region" aria-label="Suggerimento lingua">
    <div class="lang-banner-inner">
      <p>Would you prefer English?</p>
      <div class="lang-banner-actions">
        <a href="/en/" data-cta="lang_switch_banner" class="lang-banner-switch">Switch</a>
        <button id="lang-banner-close" aria-label="Chiudi">×</button>
      </div>
    </div>
  </div>
```
new:
```html
  <!-- ══════════════════════ LANGUAGE BANNER ═══════════════════ -->
  <div id="lang-banner" role="region" aria-label="Language suggestion">
    <div class="lang-banner-inner">
      <p>Preferisci l'italiano?</p>
      <div class="lang-banner-actions">
        <a href="/" data-cta="lang_switch_banner" class="lang-banner-switch">Cambia</a>
        <button id="lang-banner-close" aria-label="Close">×</button>
      </div>
    </div>
  </div>
```

old (JS, nello script finale):
```javascript
  const LANG_CURRENT = 'it';
  const LANG_ALT = 'en';
```
new:
```javascript
  const LANG_CURRENT = 'en';
  const LANG_ALT = 'it';
```

- [ ] **Step 4: Tradurre l'hero**

old:
```html
        <p class="eyebrow">📍 Iglesias · Sardegna</p>
        <h1 class="text-cream font-black leading-tight mb-4"
            style="font-size:clamp(1.9rem,4.5vw,3.2rem);letter-spacing:-.5px">
          Assistenza e cura<br>per il tuo camper a Iglesias.
        </h1>
        <p class="text-cream font-bold mb-6" style="font-size:clamp(1rem,1.8vw,1.25rem);line-height:1.6">
          Il tuo camper in mani esperte. Interventi su misura.
        </p>
        <p class="text-lt leading-relaxed mb-9" style="font-size:clamp(.9rem,1.5vw,1.05rem);line-height:1.85;max-width:480px">
          Officina specializzata in camper e van a Iglesias, vicino alla SS130.
          Assistenza tecnica completa, impianti di bordo e camperizzazioni personalizzate.
        </p>
```
new:
```html
        <p class="eyebrow">📍 Iglesias · Sardinia, Italy</p>
        <h1 class="text-cream font-black leading-tight mb-4"
            style="font-size:clamp(1.9rem,4.5vw,3.2rem);letter-spacing:-.5px">
          Camper care and service<br>in Iglesias, Sardinia.
        </h1>
        <p class="text-cream font-bold mb-6" style="font-size:clamp(1rem,1.8vw,1.25rem);line-height:1.6">
          Your camper in expert hands. Tailored service.
        </p>
        <p class="text-lt leading-relaxed mb-9" style="font-size:clamp(.9rem,1.5vw,1.05rem);line-height:1.85;max-width:480px">
          Specialist workshop for campers and vans in Iglesias, near the SS130 highway.
          Full technical service, onboard systems and custom camper conversions.
        </p>
```

- [ ] **Step 5: Tradurre i CTA dell'hero**

old:
```html
          <a href="https://wa.me/393513531335" data-cta="whatsapp_hero" class="btn btn-wa" aria-label="Scrivici su WhatsApp">
```
new:
```html
          <a href="https://wa.me/393513531335" data-cta="whatsapp_hero" class="btn btn-wa" aria-label="Message us on WhatsApp">
```

old:
```html
            Scrivici su WhatsApp
          </a>
          <a href="tel:+393513531335" data-cta="tel_hero" class="btn btn-tel">
            📞 Chiamaci: 351 353 1335
          </a>
```
new:
```html
            Message us on WhatsApp
          </a>
          <a href="tel:+393513531335" data-cta="tel_hero" class="btn btn-tel">
            📞 Call us: +39 351 353 1335
          </a>
```

- [ ] **Step 6: Tradurre la strip posizione**

old:
```html
      <p style="margin:0;font-size:.88rem;font-weight:600;color:var(--c-text);line-height:1.5">
        A pochi minuti dalla SS130 · Iglesias (SU)
      </p>
```
new:
```html
      <p style="margin:0;font-size:.88rem;font-weight:600;color:var(--c-text);line-height:1.5">
        A few minutes from the SS130 highway · Iglesias (SU), Sardinia
      </p>
```

- [ ] **Step 7: Verificare che non restino frammenti in italiano nelle sezioni tradotte finora**

Run: `sed -n '480,572p' en/index.html | grep -iE "chi siamo|dove siamo|contattaci|chiamaci|scrivici|torna in cima|navigazione principale"`
Expected: nessun output (nessuna corrispondenza).

- [ ] **Step 8: Commit**

```bash
git add en/index.html
git commit -m "Traduce navbar, switcher, banner lingua, hero e strip posizione in en/index.html"
```

---

### Task 4: Tradurre "Chi siamo" e "Servizi" in `en/index.html`

**Files:**
- Modify: `en/index.html`

- [ ] **Step 1: Tradurre la sezione Chi siamo (About us)**

old:
```html
          <span class="eyebrow">Chi siamo</span>
          <h2 class="section-title text-forest fade-up" style="margin-bottom:0">
            Competenza artigianale.<br>Gestione diretta<br>e preventivi chiari.
          </h2>
        </div>

        <!-- Corpo testo -->
        <div>
          <p class="fade-up d1" style="font-size:.93rem;line-height:1.9;margin-bottom:1.25rem;color:var(--c-text)">
            Iglesias Camper Service nasce dalla passione diretta per la vita su quattro ruote
            e da anni di esperienza pratica sul campo.
          </p>
          <p class="fade-up d2" style="font-size:.93rem;line-height:1.9;margin-bottom:1.5rem;color:var(--c-text)">
            Ogni intervento viene seguito personalmente, con attenzione ai dettagli
            e rispetto per il budget del cliente.
          </p>
          <div class="quote-block fade-up d3">
            Nessun intermediario, nessuna sorpresa in fattura.<br>
            Il preventivo è sempre concordato prima di iniziare qualsiasi lavoro.
          </div>
```
new:
```html
          <span class="eyebrow">About us</span>
          <h2 class="section-title text-forest fade-up" style="margin-bottom:0">
            Hands-on craftsmanship.<br>Direct management<br>and clear quotes.
          </h2>
        </div>

        <!-- Corpo testo -->
        <div>
          <p class="fade-up d1" style="font-size:.93rem;line-height:1.9;margin-bottom:1.25rem;color:var(--c-text)">
            Iglesias Camper Service was born from a genuine passion for life on four wheels
            and years of hands-on, practical experience.
          </p>
          <p class="fade-up d2" style="font-size:.93rem;line-height:1.9;margin-bottom:1.5rem;color:var(--c-text)">
            Every job is handled personally, with attention to detail
            and respect for the customer's budget.
          </p>
          <div class="quote-block fade-up d3">
            No middlemen, no surprises on the invoice.<br>
            The quote is always agreed before any work begins.
          </div>
```

- [ ] **Step 2: Tradurre l'intestazione della sezione Servizi**

old:
```html
        <span class="eyebrow">Cosa facciamo</span>
        <h2 class="section-title text-forest">I nostri servizi</h2>
```
new:
```html
        <span class="eyebrow">What we do</span>
        <h2 class="section-title text-forest">Our services</h2>
```

- [ ] **Step 3: Tradurre la card "Impianti di Bordo"**

old:
```html
            <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0">Impianti di Bordo</h3>
          </div>
          <p style="margin:0;color:var(--c-muted);font-size:.88rem;line-height:1.85">
            Batterie al litio, pannelli solari, inverter, presa 220V, impianti idrici.
            Installazione, aggiornamento e diagnostica completa.
          </p>
```
new:
```html
            <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0">Onboard Systems</h3>
          </div>
          <p style="margin:0;color:var(--c-muted);font-size:.88rem;line-height:1.85">
            Lithium batteries, solar panels, inverters, 220V sockets, water systems.
            Installation, upgrades and full diagnostics.
          </p>
```

- [ ] **Step 4: Tradurre la card "Igiene & Sanificazione"**

old:
```html
            <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0">Igiene &amp; Sanificazione</h3>
          </div>
          <p style="margin:0;color:var(--c-muted);font-size:.88rem;line-height:1.85">
            Pulizia e igienizzazione di cisterne (acque chiare, grigie e nere).
            Sanificazione professionale di interni, tessuti e moquette.
          </p>
```
new:
```html
            <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0">Hygiene &amp; Sanitization</h3>
          </div>
          <p style="margin:0;color:var(--c-muted);font-size:.88rem;line-height:1.85">
            Cleaning and sanitizing tanks (fresh, grey and black water).
            Professional sanitization of interiors, upholstery and carpets.
          </p>
```

- [ ] **Step 5: Tradurre la card "Falegnameria & Modifiche"**

old:
```html
            <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0">Falegnameria &amp; Modifiche</h3>
          </div>
          <p style="margin:0;color:var(--c-muted);font-size:.88rem;line-height:1.85">
            Lavori in legno per interni, modifiche strutturali e personalizzazioni —
            box e rastrelliere su misura per portare tavole da surf e SUP in viaggio.
            Realizzati su progetto concordato con il cliente.
          </p>
```
new:
```html
            <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0">Carpentry &amp; Modifications</h3>
          </div>
          <p style="margin:0;color:var(--c-muted);font-size:.88rem;line-height:1.85">
            Woodwork for interiors, structural changes and customizations —
            custom boxes and racks for carrying surfboards and SUPs while you travel.
            Built to a project agreed with the customer.
          </p>
```

- [ ] **Step 6: Tradurre la card "Manutenzione & Riparazioni"**

old:
```html
            <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0">Manutenzione &amp; Riparazioni</h3>
          </div>
          <p style="margin:0;color:var(--c-muted);font-size:.88rem;line-height:1.85">
            Tagliando, cambio olio e filtri. Interventi rapidi e preventivi trasparenti.
          </p>
```
new:
```html
            <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0">Maintenance &amp; Repairs</h3>
          </div>
          <p style="margin:0;color:var(--c-muted);font-size:.88rem;line-height:1.85">
            Servicing, oil and filter changes. Quick turnaround and transparent quotes.
          </p>
```

- [ ] **Step 7: Tradurre la card featured "Camperizzazioni su Misura"**

old:
```html
              <h3 style="font-size:1.05rem;font-weight:800;color:var(--c-cream);margin:0">Camperizzazioni su Misura</h3>
            </div>
            <p style="margin:0;font-size:.9rem;line-height:1.85;color:var(--c-forest-lt)">
              Conversione completa di furgoni, van e veicoli speciali.
              Progettazione e realizzazione dall'inizio alla fine, in base alle tue esigenze specifiche —
              anche per chi viaggia da surfista, con spazio e soluzioni dedicate al trasporto delle tavole.
            </p>
          </div>
          <div class="featured-cta">
            <p style="font-size:.88rem;color:var(--c-forest-lt);line-height:1.7;margin-bottom:1.25rem">
              Hai un furgone da trasformare, magari per portarti dietro le tavole da surf?
              Parliamone insieme — valutiamo il progetto senza impegno.
            </p>
            <a href="https://wa.me/393513531335?text=Ciao%2C%20vorrei%20informazioni%20per%20una%20camperizzazione%20su%20misura"
               data-cta="whatsapp_preventivo" class="btn btn-wa" style="max-width:280px">
              Richiedi un preventivo →
            </a>
```
new:
```html
              <h3 style="font-size:1.05rem;font-weight:800;color:var(--c-cream);margin:0">Custom Camper Conversions</h3>
            </div>
            <p style="margin:0;font-size:.9rem;line-height:1.85;color:var(--c-forest-lt)">
              Complete conversion of vans and special vehicles.
              Design and build from start to finish, based on your specific needs —
              including surfers who need space and dedicated solutions for carrying boards.
            </p>
          </div>
          <div class="featured-cta">
            <p style="font-size:.88rem;color:var(--c-forest-lt);line-height:1.7;margin-bottom:1.25rem">
              Got a van you'd like to convert, maybe to fit your surfboards?
              Let's talk — we'll assess the project with no obligation.
            </p>
            <a href="https://wa.me/393513531335?text=Hi%2C%20I%27d%20like%20info%20about%20a%20custom%20camper%20conversion"
               data-cta="whatsapp_preventivo" class="btn btn-wa" style="max-width:280px">
              Get a quote →
            </a>
```

- [ ] **Step 8: Verificare che non restino frammenti in italiano nelle due sezioni**

Run: `sed -n '573,694p' en/index.html | grep -iE "chi siamo|impianti|sanificazione|falegnameria|manutenzione|camperizzazion|preventivo"`
Expected: nessun output.

- [ ] **Step 9: Commit**

```bash
git add en/index.html
git commit -m "Traduce sezioni Chi siamo e Servizi in en/index.html"
```

---

### Task 5: Tradurre "Contatti" e "FAQ" in `en/index.html`

**Files:**
- Modify: `en/index.html`

- [ ] **Step 1: Tradurre la sezione Dove siamo/Contatti**

old:
```html
          <span class="eyebrow">Dove siamo</span>
          <h2 class="section-title text-cream fade-up" style="margin-bottom:1.25rem">
            Iglesias, Sardegna
          </h2>
          <p class="text-lt fade-up d1" style="font-size:.93rem;line-height:1.85;max-width:400px">
            A pochi minuti dalla SS130.
            Contattaci per informazioni, preventivi e appuntamenti.
          </p>
          <!-- TODO(Nicola): confermare l'elenco dei comuni serviti prima di pubblicare -->
          <p class="text-lt fade-up d1" style="font-size:.93rem;line-height:1.85;max-width:400px;margin-top:1rem">
            Serviamo camperisti e vanlifer di <strong class="text-cream">Iglesias,
            Carbonia, Domusnovas, Gonnesa e Musei</strong> — e di tutto il Sulcis Iglesiente.
          </p>
        </div>

        <!-- CTA -->
        <div class="contact-ctas fade-up d2">
          <a href="https://wa.me/393513531335" data-cta="whatsapp_contatti" class="btn btn-wa">
            💬 Scrivici su WhatsApp
          </a>
          <a href="tel:+393513531335" data-cta="tel_contatti" class="btn btn-tel">
            📞 Chiamaci: 351 353 1335
          </a>
          <a href="https://maps.google.com/?q=Az.+Agricola+Vacca+Monte+Cadelano+Villamassargia+SU"
             target="_blank" rel="noopener noreferrer" data-cta="maps_contatti" class="btn btn-maps">
            🗺️ Apri in Google Maps ↗
          </a>
```
new:
```html
          <span class="eyebrow">Where we are</span>
          <h2 class="section-title text-cream fade-up" style="margin-bottom:1.25rem">
            Iglesias, Sardinia
          </h2>
          <p class="text-lt fade-up d1" style="font-size:.93rem;line-height:1.85;max-width:400px">
            A few minutes from the SS130 highway.
            Contact us for information, quotes and appointments.
          </p>
          <!-- TODO(Nicola): confermare l'elenco dei comuni serviti prima di pubblicare -->
          <p class="text-lt fade-up d1" style="font-size:.93rem;line-height:1.85;max-width:400px;margin-top:1rem">
            We serve camper and van travelers from <strong class="text-cream">Iglesias,
            Carbonia, Domusnovas, Gonnesa and Musei</strong> — and the whole Sulcis Iglesiente area.
          </p>
        </div>

        <!-- CTA -->
        <div class="contact-ctas fade-up d2">
          <a href="https://wa.me/393513531335" data-cta="whatsapp_contatti" class="btn btn-wa">
            💬 Message us on WhatsApp
          </a>
          <a href="tel:+393513531335" data-cta="tel_contatti" class="btn btn-tel">
            📞 Call us: +39 351 353 1335
          </a>
          <a href="https://maps.google.com/?q=Az.+Agricola+Vacca+Monte+Cadelano+Villamassargia+SU"
             target="_blank" rel="noopener noreferrer" data-cta="maps_contatti" class="btn btn-maps">
            🗺️ Open in Google Maps ↗
          </a>
```

- [ ] **Step 2: Tradurre l'intestazione FAQ**

old:
```html
        <span class="eyebrow">Domande frequenti</span>
        <h2 class="section-title text-forest">FAQ — Iglesias Camper Service</h2>
```
new:
```html
        <span class="eyebrow">Frequently asked questions</span>
        <h2 class="section-title text-forest">FAQ — Iglesias Camper Service</h2>
```

- [ ] **Step 3: Tradurre le 6 domande/risposte FAQ visibili**

old:
```html
        <div class="fade-up d1">
          <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0 0 .5rem">
            In quali zone operate?
          </h3>
          <p style="margin:0;color:var(--c-text);font-size:.9rem;line-height:1.85">
            L'officina si trova a Iglesias, a pochi minuti dalla SS130.
            Serviamo tutto il Sulcis Iglesiente: Iglesias, Carbonia,
            Domusnovas, Gonnesa e Musei.
          </p>
        </div>

        <div class="fade-up d2">
          <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0 0 .5rem">
            Fate anche camperizzazioni complete di furgoni?
          </h3>
          <p style="margin:0;color:var(--c-text);font-size:.9rem;line-height:1.85">
            Sì. Progettiamo e realizziamo conversioni complete di furgoni, van e veicoli
            speciali, dall'idea alla consegna, in base alle esigenze di chi viaggia.
          </p>
        </div>

        <div class="fade-up d3">
          <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0 0 .5rem">
            Serve appuntamento o si può passare direttamente?
          </h3>
          <p style="margin:0;color:var(--c-text);font-size:.9rem;line-height:1.85">
            Meglio contattarci prima su WhatsApp o per telefono al 351 353 1335: così
            valutiamo insieme l'intervento e concordiamo giorno e ora, senza attese.
          </p>
        </div>

        <div class="fade-up d4">
          <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0 0 .5rem">
            Fate la pulizia e sanificazione delle cisterne dell'acqua?
          </h3>
          <p style="margin:0;color:var(--c-text);font-size:.9rem;line-height:1.85">
            Sì. Igienizziamo le cisterne delle acque chiare, grigie e nere e sanifichiamo
            gli interni del camper, tessuti e moquette compresi.
          </p>
        </div>

        <div class="fade-up d5">
          <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0 0 .5rem">
            Come funziona il preventivo?
          </h3>
          <p style="margin:0;color:var(--c-text);font-size:.9rem;line-height:1.85">
            Il preventivo viene sempre concordato prima di iniziare qualsiasi lavoro,
            senza sorprese in fattura. La valutazione del progetto è senza impegno.
          </p>
        </div>

        <div class="fade-up d5">
          <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0 0 .5rem">
            Lavorate solo su camper o anche su van?
          </h3>
          <p style="margin:0;color:var(--c-text);font-size:.9rem;line-height:1.85">
            Lavoriamo su camper, van e furgoni camperizzati: assistenza tecnica, impianti
            di bordo, manutenzione e allestimenti su misura.
          </p>
        </div>
```
new:
```html
        <div class="fade-up d1">
          <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0 0 .5rem">
            Which areas do you serve?
          </h3>
          <p style="margin:0;color:var(--c-text);font-size:.9rem;line-height:1.85">
            Our workshop is located in Iglesias, a few minutes from the SS130 highway.
            We serve the whole Sulcis Iglesiente area: Iglesias, Carbonia,
            Domusnovas, Gonnesa and Musei.
          </p>
        </div>

        <div class="fade-up d2">
          <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0 0 .5rem">
            Do you also do full van conversions?
          </h3>
          <p style="margin:0;color:var(--c-text);font-size:.9rem;line-height:1.85">
            Yes. We design and build complete conversions of vans and special
            vehicles, from concept to delivery, based on the needs of those who travel.
          </p>
        </div>

        <div class="fade-up d3">
          <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0 0 .5rem">
            Do I need an appointment, or can I just stop by?
          </h3>
          <p style="margin:0;color:var(--c-text);font-size:.9rem;line-height:1.85">
            It's best to contact us first on WhatsApp or by phone at +39 351 353 1335:
            that way we can discuss the job together and agree on a day and time, with no waiting around.
          </p>
        </div>

        <div class="fade-up d4">
          <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0 0 .5rem">
            Do you clean and sanitize water tanks?
          </h3>
          <p style="margin:0;color:var(--c-text);font-size:.9rem;line-height:1.85">
            Yes. We sanitize fresh, grey and black water tanks and disinfect
            camper interiors, including upholstery and carpets.
          </p>
        </div>

        <div class="fade-up d5">
          <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0 0 .5rem">
            How does the quote work?
          </h3>
          <p style="margin:0;color:var(--c-text);font-size:.9rem;line-height:1.85">
            The quote is always agreed before starting any work,
            with no surprises on the invoice. Project assessment is free of charge, with no obligation.
          </p>
        </div>

        <div class="fade-up d5">
          <h3 style="font-size:1rem;font-weight:800;color:var(--c-forest);margin:0 0 .5rem">
            Do you only work on campers, or vans too?
          </h3>
          <p style="margin:0;color:var(--c-text);font-size:.9rem;line-height:1.85">
            We work on campers, vans and camperized vans: technical service, onboard
            systems, maintenance and custom fit-outs.
          </p>
        </div>
```

- [ ] **Step 4: Verificare che non restino frammenti in italiano**

Run: `sed -n '698,812p' en/index.html | grep -iE "dove siamo|preventivo|scrivici|chiamaci|zone operate|camperizzazioni complete"`
Expected: nessun output.

- [ ] **Step 5: Commit**

```bash
git add en/index.html
git commit -m "Traduce sezioni Contatti e FAQ in en/index.html"
```

---

### Task 6: Tradurre footer, cookie banner, widget accessibilità, WhatsApp float in `en/index.html`

**Files:**
- Modify: `en/index.html`

- [ ] **Step 1: Tradurre il footer**

old:
```html
        <p style="color:#6b9e8e;font-size:.8rem;line-height:1.7;margin:0">
          © 2026 · Iglesias Camper Service<br>
          Iglesias (SU) · Sardegna
        </p>
      </div>
      <p style="color:#4a7a6a;font-size:.75rem;line-height:1.7;margin:0">
        Tutti i diritti riservati<br>
        <a href="#" id="cookie-manage-link" style="color:#4a7a6a;text-decoration:underline">Gestisci preferenze cookie</a>
      </p>
```
new:
```html
        <p style="color:#6b9e8e;font-size:.8rem;line-height:1.7;margin:0">
          © 2026 · Iglesias Camper Service<br>
          Iglesias (SU) · Sardinia, Italy
        </p>
      </div>
      <p style="color:#4a7a6a;font-size:.75rem;line-height:1.7;margin:0">
        All rights reserved<br>
        <a href="#" id="cookie-manage-link" style="color:#4a7a6a;text-decoration:underline">Manage cookie preferences</a>
      </p>
```

- [ ] **Step 2: Tradurre il banner cookie**

old:
```html
  <div id="cookie-banner" role="dialog" aria-label="Consenso cookie">
    <p>
      Usiamo cookie tecnici necessari al funzionamento del sito e, solo previo tuo consenso,
      cookie di analisi (Google Analytics) per capire come viene usato il sito.
    </p>
    <div class="cookie-actions">
      <button id="cookie-decline" class="btn-cookie btn-cookie-outline">Rifiuta</button>
      <button id="cookie-accept"  class="btn-cookie btn-cookie-solid">Accetta</button>
    </div>
  </div>
```
new:
```html
  <div id="cookie-banner" role="dialog" aria-label="Cookie consent">
    <p>
      We use technical cookies necessary for the site to work and, only with your consent,
      analytics cookies (Google Analytics) to understand how the site is used.
    </p>
    <div class="cookie-actions">
      <button id="cookie-decline" class="btn-cookie btn-cookie-outline">Decline</button>
      <button id="cookie-accept"  class="btn-cookie btn-cookie-solid">Accept</button>
    </div>
  </div>
```

- [ ] **Step 3: Tradurre il widget accessibilità**

old:
```html
  <button id="a11y-toggle" aria-label="Strumenti di accessibilità"
           aria-expanded="false" aria-controls="a11y-panel">
```
new:
```html
  <button id="a11y-toggle" aria-label="Accessibility tools"
           aria-expanded="false" aria-controls="a11y-panel">
```

old:
```html
  <div id="a11y-panel" role="dialog" aria-label="Strumenti di accessibilità" aria-modal="false">
    <span class="a11y-label">Accessibilità</span>
    <div class="a11y-row">
      <span class="a11y-row-label">Testo</span>
      <button class="a11y-btn" id="btn-fs-down" aria-label="Riduci testo">A−</button>
      <button class="a11y-btn" id="btn-fs-up"   aria-label="Aumenta testo">A+</button>
    </div>
    <div class="a11y-row">
      <button class="a11y-btn a11y-btn-hc" id="btn-hc"
              aria-pressed="false" aria-label="Attiva alto contrasto">
        ☀ Alto contrasto
      </button>
    </div>
  </div>
```
new:
```html
  <div id="a11y-panel" role="dialog" aria-label="Accessibility tools" aria-modal="false">
    <span class="a11y-label">Accessibility</span>
    <div class="a11y-row">
      <span class="a11y-row-label">Text</span>
      <button class="a11y-btn" id="btn-fs-down" aria-label="Decrease text size">A−</button>
      <button class="a11y-btn" id="btn-fs-up"   aria-label="Increase text size">A+</button>
    </div>
    <div class="a11y-row">
      <button class="a11y-btn a11y-btn-hc" id="btn-hc"
              aria-pressed="false" aria-label="Enable high contrast">
        ☀ High contrast
      </button>
    </div>
  </div>
```

- [ ] **Step 4: Tradurre il testo dinamico del bottone alto contrasto nello script**

old:
```javascript
    btnHc.textContent = on ? '🌙 Contrasto normale' : '☀ Alto contrasto';
```
new:
```javascript
    btnHc.textContent = on ? '🌙 Normal contrast' : '☀ High contrast';
```

- [ ] **Step 5: Tradurre il WhatsApp float**

old:
```html
  <a id="wa-float" href="https://wa.me/393513531335" data-cta="whatsapp_float"
     aria-label="Contattaci su WhatsApp" title="Scrivici su WhatsApp">
```
new:
```html
  <a id="wa-float" href="https://wa.me/393513531335" data-cta="whatsapp_float"
     aria-label="Contact us on WhatsApp" title="Message us on WhatsApp">
```

- [ ] **Step 6: Verificare che non restino frammenti in italiano in tutto il file**

Run:
```bash
grep -inE "riservati|Accetta|Rifiuta|accessibilità|Alto contrasto|Contrasto normale|Torna in cima|Contattaci su WhatsApp|Scrivici su WhatsApp" en/index.html
```
Expected: nessun output (0 risultati). Se emergono righe, tradurle e ripetere il comando.

- [ ] **Step 7: Ri-validare i JSON-LD dopo tutte le modifiche**

Run (stesso comando del Task 2, Step 6):
```bash
node -e "
const fs = require('fs');
const html = fs.readFileSync('en/index.html', 'utf8');
const blocks = [...html.matchAll(/<script type=\"application\/ld\+json\">([\s\S]*?)<\/script>/g)];
blocks.forEach((b, i) => { JSON.parse(b[1]); console.log('JSON-LD block', i, 'OK'); });
"
```
Expected: `JSON-LD block 0 OK` e `JSON-LD block 1 OK`.

- [ ] **Step 8: Commit**

```bash
git add en/index.html
git commit -m "Traduce footer, cookie banner, widget accessibilità e WhatsApp float in en/index.html"
```

---

### Task 7: Aggiornare `sitemap.xml` con l'entry `/en/`

**Files:**
- Modify: `sitemap.xml`

- [ ] **Step 1: Aggiungere l'URL inglese con i link hreflang reciproci**

old:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://iglesiascamperservice.com/</loc>
    <lastmod>2026-07-04</lastmod>
    <changefreq>monthly</changefreq>
    <priority>1.0</priority>
  </url>
</urlset>
```
new:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9"
        xmlns:xhtml="http://www.w3.org/1999/xhtml">
  <url>
    <loc>https://iglesiascamperservice.com/</loc>
    <lastmod>2026-07-05</lastmod>
    <changefreq>monthly</changefreq>
    <priority>1.0</priority>
    <xhtml:link rel="alternate" hreflang="it" href="https://iglesiascamperservice.com/"/>
    <xhtml:link rel="alternate" hreflang="en" href="https://iglesiascamperservice.com/en/"/>
    <xhtml:link rel="alternate" hreflang="x-default" href="https://iglesiascamperservice.com/"/>
  </url>
  <url>
    <loc>https://iglesiascamperservice.com/en/</loc>
    <lastmod>2026-07-05</lastmod>
    <changefreq>monthly</changefreq>
    <priority>1.0</priority>
    <xhtml:link rel="alternate" hreflang="it" href="https://iglesiascamperservice.com/"/>
    <xhtml:link rel="alternate" hreflang="en" href="https://iglesiascamperservice.com/en/"/>
    <xhtml:link rel="alternate" hreflang="x-default" href="https://iglesiascamperservice.com/"/>
  </url>
</urlset>
```

- [ ] **Step 2: Verificare che il file sia XML valido**

Run: `python3 -c "import xml.dom.minidom as m; m.parse('sitemap.xml'); print('OK')"`
Expected: `OK`

- [ ] **Step 3: Commit**

```bash
git add sitemap.xml
git commit -m "Aggiunge /en/ a sitemap.xml con link hreflang reciproci"
```

---

### Task 8: Verifica finale end-to-end

**Files:**
- Nessuna modifica (solo verifica); eventuali piccoli fix vanno applicati a `index.html` o `en/index.html`.

- [ ] **Step 1: Verificare la reciprocità degli hreflang tra i due file**

Run:
```bash
grep -o 'hreflang="[a-z-]*" href="[^"]*"' index.html
grep -o 'hreflang="[a-z-]*" href="[^"]*"' en/index.html
```
Expected: stessi 3 URL (it, en, x-default) identici in entrambi i file.

- [ ] **Step 2: Aprire `index.html` in un browser locale e verificare manualmente**

Run: `open index.html` (macOS) o `python3 -m http.server 8000` e visitare `http://localhost:8000/`.

Checklist manuale:
- Lo switcher "EN" in navbar è visibile e porta a `/en/`.
- Il banner lingua NON appare se il browser è in italiano (comportamento di default).
- Aprire i DevTools, eseguire `localStorage.clear()` e forzare `navigator.language` (o testare direttamente aprendo `en/index.html` con browser in italiano, che è il caso equivalente speculare) per vedere il banner apparire.
- Cookie banner, widget accessibilità (A+/A-, alto contrasto), pulsante WhatsApp flottante, animazioni scroll funzionano come prima (nessuna regressione).

- [ ] **Step 3: Aprire `en/index.html` e ripetere la checklist manuale**

Run: `open en/index.html` o visitare `http://localhost:8000/en/`.

Checklist manuale (stessa del passo 2, ma verificando):
- Lo switcher "IT" in navbar porta a `/`.
- Il banner "Preferisci l'italiano?" appare se `navigator.language` inizia con `it` e non c'è preferenza salvata.
- Tutti i testi visibili sono in inglese, nessun frammento italiano residuo.
- Telefono mostrato come `+39 351 353 1335`, link `tel:`/WhatsApp invariati.
- Link Google Maps punta ancora a Villamassargia (invariato).

- [ ] **Step 4: Verificare che il click sullo switcher/banner venga tracciato da GA4**

Aprire la Console DevTools su `index.html`, cliccare sul link "EN" in navbar, verificare che non ci siano errori JS. Il click viene già intercettato dal listener generico `[data-cta]` sul banner (`data-cta="lang_switch_banner"`); lo switcher navbar non ha `data-cta` per design (è navigazione, non un CTA di conversione) — verificare che questo comportamento sia quello atteso.

- [ ] **Step 5: Commit finale (se sono stati necessari fix durante la verifica)**

```bash
git add -A
git commit -m "Fix finali dopo verifica end-to-end versione EN"
```

(Se non sono stati necessari fix, questo step si salta.)
