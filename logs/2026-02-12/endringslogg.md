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
