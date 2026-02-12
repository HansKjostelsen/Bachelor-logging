# Endringslogg - Tirsdag 11. februar 2026

## Oversikt
ISO 9001-siden ble betydelig utvidet med responsiv fitView-funksjonalitet, klikkbare noder med navigasjon, og 9 nye undersider for ulike prosessområder. Alle noder fikk slug- og description-felt, og interaktiv hover-funksjonalitet ble implementert.

---

## Commits (kronologisk)

### 1. Oppretta ISO-9001 eksempel flow (`5bfacbd`)

**Lagt til:**
- 9 nye undersider i `Frontend/src/pages/ISO-sider/9001-Undersider/`:
  - `SalgOgKontraktshaandtering.jsx` (82 linjer)
  - `UtformingOgUtvikling.jsx` (90 linjer)
  - `Anskaffelse.jsx` (90 linjer)
  - `ProduksjonOgTjenesteLeveranse.jsx` (82 linjer)
  - `TestOgFrigivelse.jsx` (82 linjer)
  - `Leveranse.jsx` (90 linjer)
  - `KundeTilfredshet.jsx` (82 linjer)
  - `Visjon.jsx` (82 linjer)
  - `Kunder.jsx` (90 linjer)
- Hver underside har 3-4 placeholder-noder med React Flow
- Tilbake-knapp for navigasjon til hovedsiden

**Endret i `Frontend/src/pages/ISO-sider/ISO9001.jsx`:**
- Lagt til responsiv fitView-funksjonalitet med `useEffect` og window resize-lytter
- Fjernet låst zoom (`minZoom={0.7}` og `maxZoom={0.7}`)
- Lagt til `minZoom={0.1}` (senere justert til 0.3)
- Lagt til `onInit={setRfInstance}` for å lagre React Flow-instansen
- Lagt til `fitViewOptions={{ padding: 0.2 }}` (senere justert til 0.1)
- Lagt til `translateExtent` for å begrense pan-området
- Lagt til slug og description på alle 9 noder
- Node-data utvidet med navigasjonskoblinger

**Endret i `Frontend/src/components/FlowCRTNode.jsx`:**
- Implementert klikkbarhet med `useNavigate` fra react-router-dom
- Lagt til hover-logikk som bytter mellom label og description
- `handleClick` funksjon som navigerer til `/iso9001/{slug}`

**Endret i `Frontend/src/App.jsx`:**
- Lagt til 9 imports for de nye undersidene
- Lagt til 9 nye routes som kobler slug-verdier til undersider

**Endret i `Frontend/src/styles.css`:**
- Lagt til `.flowcrt-node` hover-effekter (`transform: scale(1.05)`)
- Lagt til `cursor: pointer` og transitions
- Lagt til `.back-button` styling (grønn bakgrunn, gull hover)
- Økt node-høyde fra 50px til 70px
- Lagt til `.flowcrt-description` styling

**Lagt til:**
- Docker-konfigurasjon for API, Database, og Frontend
- `API/src/api.py` (8 linjer)
- `Database/migration/` SQL-filer (V1, V2, V3)
- `Frontend/Dockerfile` og `compose.yaml`
- Diverse `.dockerignore` og konfigurasjonsfiler

### 2. Merge branch 'main' into 9001-modifikasjoner (`a902e44`)

- Merget endringer fra main-branchen inn i 9001-modifikasjoner

### 3. Merge pull request #4 - 9001-modifikasjoner (`4a378a4`)

- Merget 9001-modifikasjoner inn i main via PR #4

### 4. Sletta ISO9001Sub (`2d6ea5b`)

- Ryddet opp i filer

### 5. Merge pull request #5 - 9001-modifikasjoner (`7a0a596`)

- Merget 9001-modifikasjoner inn i main via PR #5

### 6. Merge pull request #6 - main (`4c2a501`)

- Merget main-branchen for å holde 9001-modifikasjoner oppdatert

### 7. Endret overskrift (`864cc20`)

**Endret i `Frontend/src/layout/Header.jsx`:**
- Mindre justeringer i header-komponenten

**Endret i `Frontend/src/pages/home.jsx`:**
- Oppdatert innhold på hjemmesiden

**Endret i `Frontend/src/pages/login.jsx`:**
- Lagt til ny login-side (22 linjer)

### 8. Merge pull request #7 - 9001-modifikasjoner (`82c75ff`)

- Endelig merge av alle 9001-modifikasjoner inn i main
- Løst merge-konflikt i `App.jsx` hvor både undersider og Login-funksjonalitet ble beholdt

---

## Oppsummering
- **40 filer endret**, 1331 nye linjer (+), 21 linjer slettet (-)
- ISO 9001-siden fikk responsiv fitView som tilpasser seg vindusstørrelse automatisk
- 9 nye undersider opprettet for ISO 9001-prosessområder med dedikerte React Flow-visualiseringer
- Alle noder er nå klikkbare og navigerer til respektive undersider
- Hover-funksjonalitet som viser description i stedet for label
- Implementert begrensning av pan-område med translateExtent
- CSS-forbedringer med hover-effekter, pointer cursor, og back-button styling
- Docker-konfigurasjon lagt til for hele stack (API, Database, Frontend)
- Login-funksjonalitet lagt til
