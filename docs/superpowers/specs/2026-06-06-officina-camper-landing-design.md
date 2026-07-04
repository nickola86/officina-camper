# Spec: Landing Page — Officina Camper Iglesias

## Obiettivo
Landing page a singola pagina per convertire visitatori in contatti diretti via WhatsApp o telefono. Nessun e-commerce, nessun form complesso.

---

## Dati chiave
- **Titolare:** Nicola
- **Località:** Iglesias (SU), Sardegna — Sulcis Iglesiente, vicino alla SS130
- **Telefono/WhatsApp:** +39 3513531335 → `https://wa.me/393513531335` / `tel:+393513531335`
- **Anno copyright:** 2026
- **P.IVA / Indirizzo:** da inserire nel footer

---

## Target e accessibilità
Camperisti, nomadi digitali, vanlifers, proprietari di camper datati e furgoni da camperizzare. Utenti di età mista: giovani e anziani. **Priorità assoluta a leggibilità e contrasto** — font grandi (min 18px body), tap target ≥ 48px, contrasti WCAG AA/AAA.

---

## Tono di voce
Professionale, diretto, artigianale. Caldo ma concreto. Nessun linguaggio troppo "bohémien". Nessuna contrapposizione esplicita con altre categorie commerciali.

---

## Stack tecnico
- **Un singolo file** `index.html`
- **Tailwind CSS** via CDN Play (`<script src="https://cdn.tailwindcss.com"></script>`)
- **Icone:** SVG inline (Lucide-style, disegnate a mano)
- **JavaScript vanilla** (~30–50 righe) per:
  - Pulsante WhatsApp flottante fisso in basso a destra
  - Animazioni scroll con `IntersectionObserver` (fade-in + slide-up sulle card servizi) — sottili, non aggressive
  - Smooth scroll per link interni del navbar
- Hosting: file statico, compatibile con GitHub Pages / Netlify / FTP

---

## Palette colori
| Ruolo | Valore |
|---|---|
| Sfondo principale | `#FDFBF7` (crema caldo) |
| Sfondo secondario strip | `#F0EBE0` |
| Verde foresta (primario) | `#2D4A43` |
| Verde scuro (header/footer) | `#1a2e28` |
| Verde testo chiaro | `#b8cfc9` |
| Arancio terra (accenti/CTA) | `#D97706` |
| WhatsApp | `#25D366` |
| Testo su crema | `#4a3e30` |
| Bordi card | `#e0d8c8` |

---

## Struttura sezioni (dall'alto verso il basso)

### 1. Navbar (sticky)
- Sfondo: `#2D4A43`
- Sinistra: icona SVG van arancio + testo "Officina Camper | Iglesias"
- Destra: pulsante pill WhatsApp verde (`#25D366`) — etichetta "Contattaci"

### 2. Hero — Split
**Blocco superiore (verde `#2D4A43`):**
- Eyebrow: "📍 IGLESIAS · SULCIS IGLESIENTE" in arancio, uppercase, spaziato
- H1 (~1.55rem, grassetto pesante): "Il tuo camper / in mani esperte. / Interventi su misura."
- Sottotitolo (~0.88rem, colore chiaro): descrizione dell'officina, area, servizi
- CTA primaria (full-width): bottone verde WhatsApp `#25D366` — "Scrivici su WhatsApp"
- CTA secondaria (full-width): outline bianco — "📞 Chiamaci: 351 353 1335"

**Strip inferiore (crema `#F0EBE0`):**
- Icona pin + testo: "A pochi minuti dalla SS130 · nel cuore del Sulcis Iglesiente"

### 3. Chi siamo (fondo crema)
- Eyebrow arancio: "CHI SIAMO"
- H2: "Competenza artigianale. / Gestione diretta e preventivi chiari."
- Corpo (~0.9rem, line-height 1.7): descrizione dell'officina, enfasi su gestione personale, nessun intermediario, preventivo sempre concordato prima

### 4. Servizi (fondo crema)
Eyebrow + H2. Cards in colonna singola, gap 0.6rem. Ordine:

| # | Servizio | Icona | Bordo laterale |
|---|---|---|---|
| 1 | Impianti di Bordo | ⚡ | Verde `#2D4A43` |
| 2 | Igiene & Sanificazione | 💧 | Arancio `#D97706` |
| 3 | Falegnameria & Modifiche | 🪵 | Verde `#2D4A43` |
| 4 | Manutenzione & Riparazioni | 🔧 | Arancio `#D97706` |
| 5 | Camperizzazioni su Misura | 🏕️ | Featured: bg `#2D4A43` full (testo chiaro) |

Ogni card: bordo `1.5px #e0d8c8`, border-radius 10px, bordo sinistro colorato 5px (o bg pieno per la 5a).

Animazione: fade-in + translateY(20px→0) al primo ingresso in viewport, con stagger 100ms tra le card.

### 5. Dove siamo & Contatti (fondo verde `#2D4A43`)
- Eyebrow arancio: "DOVE SIAMO"
- H2: "Villamassargia (SU) · Sardegna"
- Testo: zona SS130, Sulcis Iglesiente, disponibilità WhatsApp/telefono
- CTA primaria: "💬 Scrivici su WhatsApp" (verde `#25D366`)
- CTA secondaria: "📞 Chiamaci: 351 353 1335" (outline bianco)
- CTA terziaria: "📍 Vedi su Google Maps" (outline più sottile o link testo) → `https://maps.google.com/?q=Az.+Agricola+Vacca+Monte+Cadelano+Villamassargia+SU`

### 6. Footer (verde scurisismo `#1a2e28`)
- "© 2026 · Officina Camper Iglesias · [Indirizzo]"
- "Tutti i diritti riservati · P.IVA [da inserire]"
- Testo piccolo, colore `#6b9e8e` / `#4a7a6a`

---

## Toolbar accessibilità
Piccola barra fissa in alto a destra (o integrata nel navbar su mobile), sempre visibile:

| Controllo | Comportamento |
|---|---|
| **A-** / **A+** | Diminuisce / aumenta la dimensione base del font di 2px per click (range: 14px–24px). Agisce tramite variabile CSS `--fs-base` applicata al `<body>`. |
| **Contrasto ☀/🌙** | Toggle: aggiunge la classe `.high-contrast` al `<body>`, che sovrascrive la palette con bianco/nero/giallo ad alto contrasto (WCAG AAA). |

Persistenza: i valori vengono salvati in `localStorage` e ripristinati al caricamento.

Su mobile la toolbar è posizionata sotto il navbar, full-width, con bottoni abbastanza grandi (min 44×44px tap target).

---

## Pulsante WhatsApp flottante
- Posizione: `fixed bottom-6 right-6`, `z-index: 9999`
- Forma: cerchio 56×56px, sfondo `#25D366`, icona SVG WhatsApp bianca
- Comportamento: appare dopo 1s dal caricamento, con fade-in
- Link: `https://wa.me/393513531335`
- `aria-label="Contattaci su WhatsApp"` per accessibilità

---

## Animazioni scroll
- Trigger: `IntersectionObserver` con `threshold: 0.15`
- Effetto: `opacity 0→1` + `transform translateY(20px→0)`, duration 0.5s ease-out
- Target: card servizi (con stagger), sezioni Chi siamo e Contatti
- Nessuna animazione sull'above-the-fold (hero + navbar)

---

## Campi placeholder nel footer
- Indirizzo fisico (da inserire)
- P.IVA (da inserire)

---

## Fuori scope
- Form di contatto (solo CTA dirette)
- Google Maps embed
- Analytics/tracking
- CMS o build pipeline
