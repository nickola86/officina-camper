# Spec: Versione inglese del sito (IT/EN)

## Obiettivo
Rendere il sito disponibile anche in inglese per intercettare camperisti/vanlifer stranieri in vacanza in Sardegna, massimizzando l'indicizzazione SEO per entrambe le lingue (pagine separate + `hreflang`, non toggle client-side).

---

## Architettura file

- `index.html` — versione italiana, invariata nella struttura, resta alla root (lingua di default, `lang="it"`).
- `en/index.html` — versione inglese, nuova, stessa struttura HTML/CSS/JS di `index.html`, contenuti tradotti.
- Nessuna tooling di build: due file HTML mantenuti a mano in parallelo, coerente con l'architettura attuale (file statico, nessun CMS/build step). Le modifiche future ai contenuti vanno replicate manualmente in entrambi i file.
- Asset condivisi (`assets/hero-van.png`, `/favicon.svg`) referenziati con path assoluti (`/assets/...`, `/favicon.svg`) in `en/index.html` per funzionare correttamente da sottocartella.
- Hosting statico esistente (GitHub Pages/Netlify/FTP): `en/index.html` risolve automaticamente su `https://iglesiascamperservice.com/en/`.

---

## SEO / metadati

Su **index.html (IT)**, aggiungere nel `<head>`:
```html
<link rel="alternate" hreflang="it" href="https://iglesiascamperservice.com/">
<link rel="alternate" hreflang="en" href="https://iglesiascamperservice.com/en/">
<link rel="alternate" hreflang="x-default" href="https://iglesiascamperservice.com/">
<meta property="og:locale" content="it_IT">
<meta property="og:locale:alternate" content="en_US">
```

Su **en/index.html (EN)**, speculare:
```html
<html lang="en" ...>
<link rel="canonical" href="https://iglesiascamperservice.com/en/">
<link rel="alternate" hreflang="it" href="https://iglesiascamperservice.com/">
<link rel="alternate" hreflang="en" href="https://iglesiascamperservice.com/en/">
<link rel="alternate" hreflang="x-default" href="https://iglesiascamperservice.com/">
<meta property="og:locale" content="en_US">
<meta property="og:locale:alternate" content="it_IT">
```

- `title`, `meta description`, `og:title`, `og:description` tradotti sulla pagina EN.
- I due JSON-LD (`AutoRepair`, `FAQPage`) duplicati e tradotti in `en/index.html`. Nome attività (`Iglesias Camper Service`), telefono, `hasMap` invariati. `areaServed` (Iglesias, Carbonia, Domusnovas, Gonnesa, Musei) invariato (toponimi).
- Placeholder ancora da compilare (`streetAddress`, `postalCode`, `openingHours`) restano placeholder tradotti in inglese (es. `[ENTER STREET ADDRESS]`, `[ENTER POSTAL CODE]`, `[ENTER HOURS, e.g. Mon-Fri 08:30-18:00]`) — non inventare dati reali.
- `sitemap.xml`: aggiungere `<url>` per `https://iglesiascamperservice.com/en/` (stesso `lastmod`/`changefreq`/`priority` della home) con collegamento `hreflang` reciproco tramite namespace `xhtml` standard per sitemap multilingua.
- `robots.txt`: nessuna modifica (`Allow: /` copre già `/en/`).

---

## Selettore di lingua (navbar)

- Link testuale "EN" (su `index.html`) / "IT" (su `en/index.html`), posizionato accanto al CTA WhatsApp header, sempre visibile anche su mobile (a differenza dei nav-link `hidden md:flex`).
- Link semplice `<a href="/en/">EN</a>` / `<a href="/">IT</a>`, nessun JS necessario per il funzionamento base.
- Al click, salva la scelta in `localStorage` (`oc-lang-pref`) per sopprimere il banner di suggerimento lingua in futuro.

---

## Banner suggerimento lingua (rilevazione browser)

Per rispettare le linee guida SEO di Google (no redirect automatico basato su lingua/geolocalizzazione, che confonde i crawler e penalizza l'indicizzazione di entrambe le versioni):

- Barra sottile in cima alla pagina, sotto la navbar sticky (non in basso, per non sovrapporsi a cookie banner, WhatsApp float e widget accessibilità che occupano già quello spazio).
- Mostrata solo se: `navigator.language` non corrisponde alla lingua della pagina corrente **E** l'utente non ha già una scelta salvata in `localStorage` (`oc-lang-pref` non impostato, né dismissione precedente `oc-lang-banner-dismissed`).
- Contenuto: messaggio breve + bottone di switch (es. "Would you prefer English? [Switch]" su `index.html`; "Preferisci l'italiano? [Cambia]" su `en/index.html`) + pulsante di chiusura `×`.
- Il bottone di switch ha `data-cta="lang_switch_banner"` per rientrare nel tracking GA4 generico già esistente (listener su `[data-cta]`), nessuna nuova logica di tracking da scrivere.
- Chiusura con `×` salva `oc-lang-banner-dismissed=1` e non ripropone il banner nelle visite successive.
- Nessun redirect automatico in nessun caso.

---

## Scope contenuti da tradurre

Tutto il testo visibile e i metadati SEO, sezione per sezione:

1. `<title>`, meta description, og:title/description
2. JSON-LD `AutoRepair` (description) e `FAQPage` (6 domande/risposte)
3. Navbar: voci menu (Chi siamo→About us, Servizi→Services, Dove siamo→Location, FAQ→FAQ), CTA header (Contattaci→Contact us)
4. Hero: eyebrow, H1, sottotitolo, paragrafo, CTA (WhatsApp/telefono)
5. Strip posizione
6. Sezione Chi siamo (heading, corpo testo, quote block)
7. Sezione Servizi: eyebrow, H2, le 5 card (titoli+testo), CTA featured (testo + messaggio precompilato `?text=` su wa.me)
8. Sezione Dove siamo/Contatti: eyebrow, H2, paragrafi, elenco comuni serviti, CTA (WhatsApp/telefono/Maps — link Maps invariato)
9. FAQ visibili (le stesse 6 domande del JSON-LD)
10. Footer: copyright, "Tutti i diritti riservati"→"All rights reserved", link gestione cookie
11. Cookie banner: testo + bottoni (Rifiuta/Accetta→Decline/Accept)
12. Widget accessibilità: aria-label, "Testo"→"Text", "Alto contrasto"/"Contrasto normale"→"High contrast"/"Normal contrast"
13. WhatsApp float: aria-label/title

Dettagli concordati:
- Telefono: testo visualizzato in formato internazionale `+39 351 353 1335` su `en/index.html` (href `tel:+393513531335` invariato). WhatsApp link invariato (`wa.me/393513531335`), solo il messaggio precompilato tradotto.
- Toponimi (Iglesias, Carbonia, Domusnovas, Gonnesa, Musei, "Iglesias Camper Service") invariati; dove utile per chiarezza a un pubblico straniero, aggiungere "Sardinia, Italy" (es. eyebrow hero: "📍 Iglesias · Sardinia, Italy").
- Indirizzo reale e link Google Maps (Villamassargia) invariati, come da vincolo esistente (vedi memoria progetto: i testi restano uniformati su "Iglesias", il pin Maps punta alla vera posizione).
- GA4: stesso `GA_MEASUREMENT_ID` (`G-W8RP91ZB7B`) su entrambe le pagine — stesso sito/attività, il `page_location` già distingue la lingua tramite il path.

---

## Testing / verifica (nessuna build pipeline)

- Apertura manuale di entrambi i file in browser: verifica navigazione, CTA, banner lingua (simulando `navigator.language` via devtools), cookie banner, widget accessibilità, WhatsApp float.
- Validazione sintattica dei blocchi JSON-LD (`AutoRepair`, `FAQPage`) su `en/index.html`.
- Verifica reciprocità dei tag `hreflang` tra le due pagine e coerenza con `sitemap.xml`.

---

## Fuori scope
- Nessuna terza lingua.
- Nessun redirect server-side o geolocalizzazione IP.
- Nessuna modifica ai dati reali placeholder (P.IVA, indirizzo, orari) — restano da compilare da Nicola.
- Nessun form di contatto, nessun embed Maps, nessun CMS/build pipeline (invariati rispetto allo spec originale).
