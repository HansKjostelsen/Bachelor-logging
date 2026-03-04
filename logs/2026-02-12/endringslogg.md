# Endringslogg - Onsdag 12. februar 2026

## Oversikt
FlowCRT-systemet ble kraftig utvidet med omfattende node-funksjonalitet, inkludert redigerbare noder, multi-directional handles, og 5 nye ISO 9001-undersider. Totalt ble 39 filer modifisert eller opprettet med over 3100 linjer kode lagt til.

---

## Commits (kronologisk)

### 1. Node endringer (`711b83f`)

**Hovedfokus:** Utvidelse av node-funksjonalitet med redigerbarhet og fleksible handles.

**Endret i `Frontend/src/components/FlowCRTNode.jsx`:**
- Lagt til dobbeltklikk-funksjonalitet for å redigere nodenavn via `window.prompt()`
- Implementert slette-knapp som vises ved hover (kun for editable nodes)
- Lagt til multi-directional handles (8 handles totalt):
  - Standard-modus: 2 handles (left-target, right-source)
  - allHandles-modus: 8 handles (top, bottom, left, right - hver med target og source)
- `handleDoubleClick` med `e.stopPropagation()` for å hindre navigasjon under redigering
- `handleDelete` med `window.confirm()` sikkerhetsdialog
- Callbacks: `data.onLabelChange(id, newLabel)` og `data.onDelete(id)`
- Kontroll via `data.editable`, `data.allHandles`, og `data.showHandles` flags

**Ny fil: `Frontend/src/components/LabelNode.jsx`:**
- Enkel label-komponent uten handles
- Kun for visning av tekst (f.eks. overskrifter i flows)
- Minimalistisk design med `.label-node` CSS-klasse

**Opprettet 5 nye undersider:**
- `Frontend/src/pages/ISO-sider/9001-Undersider/Dokumentstyring.jsx` (188 linjer, 12 noder)
- `Frontend/src/pages/ISO-sider/9001-Undersider/InternkontrollOgHMS.jsx` (197 linjer, 15 noder)
- `Frontend/src/pages/ISO-sider/9001-Undersider/Leverandoerer.jsx` (197 linjer, 15 noder)
- `Frontend/src/pages/ISO-sider/9001-Undersider/MaalingOgAnalyse.jsx` (188 linjer, 12 noder)
- `Frontend/src/pages/ISO-sider/9001-Undersider/StrategiOgStyring.jsx` (197 linjer, 17 noder)

Hver underside har realistic prosessnoder som:
- **Dokumentstyring:** Identifisere behov, Opprette, Gjennomgå, Godkjenne, Distribuere, Oppdatere, Arkivere, Avvikshåndtering
- **Internkontroll og HMS:** Planlegge, Identifisere farer, Risikovurdering, Tiltak, Implementere, Overvåke, Rapportere, Forbedre
- **Leverandører:** Identifisere behov, Søk, Kvalifisering, Evaluering, Kontrakt, Bestilling, Mottak, Evaluere ytelse, Oppfølging
- **Måling og Analyse:** Definere mål, Samle data, Analysere, Rapportere, Identifisere avvik, Tiltak, Verifisere, Lukke loop
- **Strategi og Styring:** Visjon/Misjon, Strategisk planlegging, Ressursallokering, KPI, Implementering, Overvåking, Gjennomgang, Justering, Kommunikasjon

**Oppdatert eksisterende undersider (9 stk):**
- `Anskaffelse.jsx`: Oppgradert fra placeholders til 15+ realistic noder
- `KundeTilfredshet.jsx`: Utvidet med feedback-loops og målingsnoder
- `Kunder.jsx`: Forbedret med kundehåndteringsprosesser
- `Leveranse.jsx`: Detaljert leveranseprosess med kvalitetskontroll
- `ProduksjonOgTjenesteLeveranse.jsx`: Utvidet produksjonsflow
- `SalgOgKontraktshaandtering.jsx`: Forbedret salgs- og kontraktsprosess
- `TestOgFrigivelse.jsx`: Detaljert test- og godkjenningsprosess
- `UtformingOgUtvikling.jsx`: Utvidet utviklingsprosess med designfaser
- `Visjon.jsx`: Forbedret visjonsprosess med strategiske elementer

**Endret i `Frontend/src/pages/ISO-sider/ISO9001.jsx`:**
- Lagt til 5 nye noder for de nye undersidene:
  - `Dokumentstyring` (position: { x: 700, y: 200 })
  - `Internkontroll og HMS` (position: { x: 1050, y: 200 })
  - `Leverandører` (position: { x: 1400, y: 200 })
  - `Måling og Analyse` (position: { x: 1750, y: 200 })
  - `Strategi og Styring` (position: { x: 2100, y: 200 })
- Totalt 14 noder i hovedvisningen
- Hver node har slug, description, og `draggable: false`

**Endret i `Frontend/src/App.jsx`:**
- Lagt til 5 imports for de nye undersidene
- Lagt til 5 nye routes:
  - `/iso9001/dokumentstyring`
  - `/iso9001/internkontroll-og-hms`
  - `/iso9001/leverandoerer`
  - `/iso9001/maaling-og-analyse`
  - `/iso9001/strategi-og-styring`
- Lagt til Register-side import og route (`/register`)

**Endret i `Frontend/src/styles.css`:**
- Massiv utvidelse med 659+ nye linjer CSS
- Node-styling forbedringer
- Delete-button styling (`.node-delete-btn`)
- Label-node styling
- Multi-directional handle-styling
- Form-styling for login/register
- Forbedret responsivitet
- Nye farger og animasjoner

**API-endringer:**
- `API/src/Main.py` opprettet (31 linjer) - Flask API med database-integrasjon
- `API/src/app_config.py` (11 linjer) - Konfigurasjon
- `API/src/database.py` (19 linjer) - Database connection pool
- `API/.env` lagt til med environment variables
- `API/requirements.txt` oppdatert med flere avhengigheter
- `API/compose.yaml` oppdatert med environment variables

**Database-endringer:**
- `Database/compose.yaml` oppdatert med nye environment variables

**Andre endringer:**
- `Frontend/src/pages/Register.jsx` opprettet (38 linjer)
- `Frontend/src/pages/login.jsx` oppdatert med forbedret styling
- `Frontend/src/pages/about.jsx` forbedret med mer innhold
- `Frontend/src/pages/FAQ.jsx` utvidet med flere spørsmål
- `Frontend/src/pages/home.jsx` oppdatert med forbedret layout
- `Frontend/src/layout/Header.jsx` oppdatert med login/register-lenker
- Nye bilder: `FlowCRT_utenBK.png`, `Gruppe_Blide.jpg`

### 2. Merge branch 'main' into 9001-modifikasjoner (`6bdf09d`)

- Merget endringer fra main-branchen inn i 9001-modifikasjoner

### 3. Liten endring (`3aeef14`)

**Endret i `Frontend/src/styles.css`:**
- `.flow-wrapper` høyde økt fra 500px til 750px
- Gir mer plass til de større og mer komplekse flowdiagrammene

---

## Begrunnelse

Arbeidet denne dagen var preget av et bevisst valg om å bygge ut systemet i bredden og dybden samtidig, noe som kan virke ambisiøst, men som var nødvendig for å gi prosjektet et reelt og demonstrerbart innhold.

**Redigerbare noder med dobbeltklikk og slette-knapp** ble prioritert fordi FlowCRT skal være et verktøy brukere faktisk kan tilpasse. Uten mulighet til å redigere og slette noder ville applikasjonen bare vært en statisk visning, ikke et interaktivt system. Bruk av `window.prompt()` og `window.confirm()` er bevisst enkle løsninger som fungerer godt for en prototype og unngår unødvendig kompleksitet på dette stadiet.

**Multi-directional handles** (8 retninger) ble innført fordi ISO-prosesser ikke alltid følger et lineært venstre-til-høyre-mønster. Noen prosesser har tilbakekoblinger, parallelle løp eller hierarkiske relasjoner. Med kun to handles (left-target og right-source) ville mange prosessdiagrammer blitt kunstig forenklet. Det er likevel beholdt en standardmodus med to handles for de tilfellene der enkelthet er tilstrekkelig.

**LabelNode** ble opprettet som en enkel komponent uten handles fordi det ble klart at ikke alle elementer i et flowdiagram skal være koblingsbare. Overskrifter og grupperingselementer trenger synlighet, ikke interaktivitet. En egen komponenttype holder koden ryddig og separerer ansvar.

**5 nye ISO 9001-undersider** ble laget for å dekke prosessområder som manglet. Et system som bare dekker halvparten av ISO 9001 er vanskelig å vise frem som et komplett produkt. Med 14 prosessområder totalt er dekningen nå bred nok til å gi et reelt inntrykk av hva FlowCRT kan brukes til.

**Oppgradering av de 9 eksisterende undersidene** fra placeholder-noder til realistiske prosessnoder var viktig av samme grunn: placeholder-data er meningsløst for en sensor eller en bruker som skal forstå hva systemet gjør. Noder med faktiske prosessteg gjør det mulig å vurdere om applikasjonen faktisk løser det den er ment å løse.

**Flask API med database-integrasjon** ble startet fordi en ren frontend-applikasjon uten bakendstøtte ikke er en fullverdig løsning. Selv om API-et på dette tidspunktet er enkelt, legger det grunnlaget for brukerautentisering og persistent datalagring. Register-siden henger sammen med dette og er første steg mot en komplett brukerflyt.

**Økt flow-wrapper høyde fra 500px til 750px** var en praktisk nødvendighet: de nye, større flowdiagrammene med 12–17 noder passet ikke innenfor den opprinnelige høyden og ville blitt kuttet av eller presset uhensiktsmessig tett sammen.

**CSS-utvidelsen** var en konsekvens av all ny funksjonalitet. Delete-knapper, label-noder, flerskjermslayout, form-styling og animasjoner måtte alle ha tilhørende stil for å gi en helhetlig brukeropplevelse. Det ble valgt å samle CSS i én fil (`styles.css`) fremfor komponentlokale stilark, noe som gir enkel oversikt på dette stadiet av prosjektet.

Samlet sett representerer denne dagen et vesentlig sprang i modenhet for FlowCRT, fra et skjelett til et system med reelt innhold, interaktivitet og en begyndende backend-arkitektur.

---

## Oppsummering
- **39 filer endret**, 3186+ nye linjer lagt til, 275 linjer slettet
- Redigerbare noder med dobbeltklikk-funksjonalitet og slette-knapper
- Multi-directional handles (8 retninger) for fleksible node-koblinger
- LabelNode-komponent for tekst uten handles
- 5 nye ISO 9001-undersider med totalt 71 nye noder
- Oppgradert alle 9 eksisterende undersider fra placeholders til realistic prosessnoder
- Totalt 14 prosessområder i ISO 9001 hovedvisningen
- Flask API med database-integrasjon
- Register-funksjonalitet lagt til
- Massiv CSS-utvidelse med 659+ nye linjer
- Flow-wrapper høyde økt til 750px for bedre visning
