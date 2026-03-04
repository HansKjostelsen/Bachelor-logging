# Endringslogg - Torsdag 5. februar 2026

## Oversikt
React Flow ble integrert i ISO 9001-siden med custom noder, edges og ny mappestruktur for ISO-sidene.

---

## Commits (kronologisk)

### 1. Importert Flow, og endret ISO sider (`bf59251`)

**Lagt til:**
- `Frontend/src/components/FlowCRTNode.jsx` (15 linjer) - Ny custom React Flow-node med:
  - Mork gronn boks (`#013220`) med hvit tekst
  - Venstre handle (input) og hoyre handle (output)
  - Hover-effekt som bytter til gull (`#DDB771`)
- `Frontend/src/pages/ISO-sider/ISO9001.jsx` (85 linjer) - Helt ny ISO 9001-side med React Flow:
  - 3 startnoder (Node 1, Node 2, Node 3) med hardkodede posisjoner
  - 1 edge med pil mellom Node 1 og Node 2
  - `fitView` aktivert
  - Interaktive callbacks for node/edge-endringer og nye koblinger
- `Frontend/public/FlowCRT_1-1_mindre.jpg` - Logo-bilde

**Endret:**
- `Frontend/src/App.jsx` - Oppdatert imports til ny mappestruktur (`pages/ISO-sider/`)
- `Frontend/index.html` - Tittel-endring
- `Frontend/package.json` / `package-lock.json` - Lagt til `@xyflow/react` som avhengighet

**Flyttet:**
- `Frontend/src/pages/ISO14001.jsx` -> `Frontend/src/pages/ISO-sider/ISO14001.jsx`
- `Frontend/src/pages/ISO27001.jsx` -> `Frontend/src/pages/ISO-sider/ISO27001.jsx`

**Fjernet:**
- `Frontend/src/pages/ISO9001.jsx` (gammel placeholder med 12 linjer)

**Lagt til i `Frontend/src/styles.css` (54 nye linjer):**
- `.flowcrt-node` - Styling for React Flow-noder (bakgrunn, border, skygge, hover)
- `.react-flow__handle` - Styling for tilkoblingspunkter (hvit med mork border)
- `.ISO-side` - Padding for ISO-sider (40px)
- `.ISO-tittel` - Overskrift-styling (28px, mork gronn)
- `.flow-wrapper` - Container for React Flow (750px hoy, border, avrundede hjorner, gra bakgrunn)
- Bakgrunnsfarge endret fra `#f6f8fa` til `#ffff`
- Header padding endret fra `0 0 1rem 0` til `1rem 1rem 1rem 0`

### 2. Merge pull request #2 - 9001-modifikasjoner (`15756d2`)

- Merget alle React Flow-endringene fra `9001-modifikasjoner`-branchen inn i main

---

## Begrunnelse

Valget om å ta i bruk **React Flow** (`@xyflow/react`) som bibliotek for prosessvisualisering var gjennomtenkt av flere grunner. React Flow er spesialbygd for interaktive node-baserte diagrammer og håndterer tilstand, drag-and-drop og kantkobling ut av boksen. Å bygge tilsvarende funksjonalitet fra bunnen av ville krevd betydelig mer tid, og biblioteket er godt dokumentert med aktiv vedlikehold.

Opprettelsen av en dedikert **`FlowCRTNode`-komponent** i stedet for å bruke React Flows innebygde standardnoder var et bevisst valg for å sikre konsistent visuell identitet. Ved å kapsle inn styling og handles i én komponent blir det enkelt å gjenbruke og videreutvikle noden uten å berøre selve flyt-logikken.

Flyttingen av ISO-sider til en dedikert **`ISO-sider/`-undermappe** under `pages/` var et strukturelt grep for å holde koden oversiktlig etter hvert som prosjektet vokser. Siden det planlegges flere ISO-standarder (9001, 14001, 27001 m.fl.), gir en egen mappe bedre navnerom og gjør det lettere å navigere i kodebasen.

Bruk av **branches og pull requests** (branch `9001-modifikasjoner`, PR #2) er god utviklingspraksis selv i et lite prosjekt. Det gir en naturlig sjekkpunkt for å vurdere endringer samlet før de merges inn i main, og bidrar til en ryddig git-historikk som er enkel å følge i etterkant.

De spesifikke **fargevalgene** (mørk grønn `#013220` og gull `#DDB771`) er bevisste profileringsvalg for FlowCRT som prosjekt. Hover-effekten med gull gir tydelig visuell tilbakemelding til brukeren om at en node er klikkbar, noe som er viktig for brukervennlighet i et verktøy der noder er det primære interaksjonselementet.

---

## Oppsummering
- **React Flow** (`@xyflow/react`) ble lagt til som avhengighet og integrert i ISO 9001-siden
- **FlowCRTNode** custom komponent ble opprettet med handles for tilkoblinger
- **ISO-sider** ble flyttet til egen `ISO-sider/`-undermappe
- **styles.css** fikk 54 nye linjer med styling for React Flow-noder, handles, og ISO-sidelayout
- PR #2 (`9001-modifikasjoner`) ble merget inn i main
