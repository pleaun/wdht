# Tracking the Transition Away Tracker — Project Guide for Claude Code

## Vad är det här projektet?

En interaktiv världskarta som visar vilka länder som anslutit sig till Coalition of the Willing för klimatåtgärder. Kartan animeras, länder färgas gröna/röda/gula och en infopanel visas vid klick.

**Live-URL:** https://pleaun.github.io/wdht/TrackingTheTransition/tracking-the-transition-dashboard.html  
**GitHub-repo:** github.com/pleaun/wdht (mapp: `TrackingTheTransition/`)  
**Huvud-fil:** `tracking-the-transition-dashboard.html` — ALLT är i denna enda fil (HTML + CSS + JS inline)

---

## Workflow

1. Redigera `tracking-the-transition-dashboard.html` via Claude Code
2. Filen deployas automatiskt till GitHub Pages via PostToolUse-hook vid varje Edit/Write
3. Bumpa alltid versionsnumret: `const BUILD = "v0.XYZ · Mmm DD";` (finns ca rad 357)
4. Testa på live-URL:en — filen kan inte testas lokalt pga CORS (Google Sheets blockeras från file://)

---

## Tech stack

- **Canvas2D** — kartrendering via offscreen canvases (offBase, offGreen)
- **D3.js v7** — NaturalEarth1-projektion, geoCentroid, geoBounds, geoPath
- **TopoJSON** — inlinead i filen (~300kB), `objects.countries`
- **SVG overlay** — önationer (Tuvalu m.fl.) och stadmarkerare (Santa Marta)
- **Google Sheets CSV** — live-data, ingen backend

---

## Data — Google Sheets

**Sheet-URL (country data):**
```
https://docs.google.com/spreadsheets/d/e/2PACX-1vR0b53-bH6EW2fJnlELIFMnlBz8qwJnnmylQFEkJUpIHp97gKIEv0IlE8FfSwZG8_nsAcAWI4bZ7NvU/pub?gid=636099964&single=true&output=csv
```

**Sheet-URL (copy/texter):**
```
https://docs.google.com/spreadsheets/d/e/2PACX-1vR0b53-.../pub?gid=1641561421&single=true&output=csv
```

### Kolumnmappning (0-indexerad, rad 5+ är länder)

| Index | Kolumn | Innehåll |
|-------|--------|----------|
| 0 | A | Landsnamn (matchar TopoJSON) |
| 1 | B | Status: green / red / yellow / blue |
| 2 | C | Display-namn (visas i UI, tomt = använd A) |
| 7 | H | Fossil Fuel Treaty (Yes/No) |
| 9 | J | BOGA (Core/Associate/No) |
| 11 | L | PPCA (Yes/No) |
| 13 | N | CETP (Yes/No) |
| 15 | P | Transition Away Roadmap (Published/In Progress/etc) |
| 16 | Q | Transition Away Roadmap — länk |
| 17 | R | NDC-status (Submitted/Non-party/etc) |
| 18 | S | NDC — länk |
| 19 | T | NDC — förklaringstext |
| 21–26 | V–AA | Oil/Coal/Gas produktion (share%, CAGR%) |
| 27 | AB | New oil & gas licenses |
| 28 | AC | New licenses — not |
| 31–36 | AF–AK | Oil/Coal/Gas konsumtion (share%, growth%) |
| 37 | AL | Fossil-free electricity (%) |
| 38 | AM | Fossil-free electricity förändring (%) |
| 39 | AN | Grid carbon intensity |
| 40 | AO | Grid intensity trend |
| 41 | AP | Fossil fuel subsidies (% av BNP) |
| 42 | AQ | Subsidies trend |
| 43 | AR | Koldioxidpris (€/tCO₂) |
| 44 | AS | Carbon price rating |
| 45 | AT | Policy note (fritext) |

**Viktigt:** Google Sheets CSV klipper tomma trailing-celler per rad. Om energy-data inte syns: kontrollera att det inte finns tomma celler mellan sista ifyllda kolumn och energikolumnerna.

---

## Färgpalett (WDHT brand)

| Namn | Hex | Används till |
|------|-----|--------------|
| Planet Blue | `#33367c` | Texter, rubriker, landsnamn |
| Love Green | `#19cd9b` | Gröna länder, ikoner |
| Pledge Red | `#de2251` | Röda länder (blockerare) |
| Warning Yellow | `#ffbb00` | Gula länder (observatörer) |
| Planet 20 | `#eaeaf1` | Grå länder (ej anslutna) |
| Planet 50 | `#8586b0` | Sekundär text, "See more"-länk |
| Border Green | `#076b50` | Gröna länders kanter (mörkare) |
| Idea Blue 60 | `#66cef0` | Akcentfärg |

---

## Viktiga designbeslut

- **Landsnamn:** 7px, font-weight 300, Planet Blue. Leader lines för länder med projArea < 1800px². Stora länder: label stannar vid centroid utan repulsion.
- **Centroid-overrides:** USA, Frankrike, Portugal m.fl. har hårdkodade centroider (utomeuropeiska territorier drar annars iväg labeln).
- **Alaska-antimeridian:** `stitchWrappedPaths()` tar bort Path2D-sub-paths vars genomsnittliga x > 75% av canvasbredden.
- **Önationer:** Renderas som SVG-prickar (inte i TopoJSON). Tuvalu har pulsring + label alltid synlig, centrerad under pricken.
- **Santa Marta:** Stadmarkering med blob + etikett, alltid synlig.
- **Mobilläge:** Horisontell pan endast, ingen pinch-zoom. mapScale = 1.0 fast.
- **DRAFT-vattenstämpel:** Synlig i infopanelen tills den tas bort.
- **Animation:** ~5.5 sekunder totalt, Colombia startar, sedan slumpmässig ordning.

---

## Infopanel — sektioner i ordning

1. **TRANSITION AWAY POLICY STATUS** — Joined (hjärta) / Blocking (varning) / Observer (öga) / Not yet joined
2. **COMMITMENTS** — FFT, BOGA, PPCA, CETP
3. **TRANSITION PLANNING** — Transition Away Roadmap + NDC (med status-text och ↗-ikon)
4. **FOSSIL FUEL PRODUCTION**
5. **FOSSIL FUEL CONSUMPTION**
6. **ENERGY SYSTEM**
7. **PUBLIC FINANCING**
8. **NEWS** — Live via RSS

Status-ikoner: ✓ grön (Published/In Progress/Submitted/Yes), ✗ röd (Not initiated/No commitment/Non-party/No), ? gul (Overdue/Pending)

---

## Deploy

En PostToolUse-hook i `~/.claude/settings.json` pushar automatiskt ändringar till GitHub via `gh api`. Filen måste heta `tracking-the-transition-dashboard.html` för att hooken ska triggas.

GitHub Pages uppdateras inom ~30 sekunder efter push.

---

## Island nations (SVG-overlay)

```
Antigua and Barbuda, Kiribati, Maldives, Marshall Islands,
Micronesia, Palau, Saint Kitts and Nevis, Saint Lucia,
Singapore, Tuvalu, Vatican - Santa Sede
```

---

## Känt att tänka på

- Filen kan inte öppnas direkt i browser (file://) för testning — CORS blockerar Google Sheets
- Greenland räknas INTE i country counter
- Röda och gula länder räknas INTE i counter
- TopoJSON-landsnamn matchar inte alltid sheet-namn → se `NAME_MAP` i koden
- CSV-parsern hanterar multi-line quoted fields (celler med radbrytningar)
