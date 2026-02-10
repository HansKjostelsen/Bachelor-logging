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

## Oppsummering
- **React Flow** (`@xyflow/react`) ble lagt til som avhengighet og integrert i ISO 9001-siden
- **FlowCRTNode** custom komponent ble opprettet med handles for tilkoblinger
- **ISO-sider** ble flyttet til egen `ISO-sider/`-undermappe
- **styles.css** fikk 54 nye linjer med styling for React Flow-noder, handles, og ISO-sidelayout
- PR #2 (`9001-modifikasjoner`) ble merget inn i main
