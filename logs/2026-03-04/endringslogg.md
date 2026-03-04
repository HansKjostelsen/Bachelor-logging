# Endringslogg - Onsdag 4. mars 2026

## Oversikt
Dagen ble brukt på å innføre et skille mellom statiske og dynamiske ISO 9001-undersider, implementere rollebasert redigering slik at kun administratorer kan endre flowdiagrammer, og koble flow-lagring direkte mot API-et via `useFlowPage`. I tillegg fikk `FlowCRTNode`-komponenten støtte for å skjule handles enkeltvis, noe som gjør det mulig å styre visuell flyt på hoved-ISO9001-siden.

---

## Commits (kronologisk)

### 1. Reorganisering og ny `StaticFlowPage`-komponent (`247ac8d`)

**Tidspunkt:** 12:17

**Mappestruktur - alle eksisterende undersider flyttet:**
- `9001-Undersider/*.jsx` -> `9001-Undersider/Dynamiske/*.jsx` (12 filer omdøpt/flyttet)
- `useFlowPage.js` flyttet fra `9001-Undersider/` til `Frontend/src/layout/`
- Ny mappe `9001-Undersider/Statiske/` opprettet

**Ny fil: `Frontend/src/pages/ISO-sider/9001-Undersider/Statiske/StaticFlowPage.jsx`**
- Gjenbrukbar komponent for ISO-sider med faste, predefinerte noder og kanter
- Tar inn `pageTitle`, `initialNodes` og `initialEdges` som props
- Ingen redigering - noder er visningsbare men ikke modifiserbare
- Tilbake-knapp navigerer til `/iso9001`
- `fitView` med responsivt resize-lytter, identisk mønster som `FlowPageTemplate`

**Ny fil: `Frontend/src/pages/ISO-sider/9001-Undersider/Dynamiske/StrategiOgStyring.jsx`**
- Første implementering av en faktisk underside med predefinerte noder og kanter
- 11 noder: 6 styrende dokumenter i rad, 2x2 tom-grid (bunn venstre), Kunder og Leverandører (høyre)
- 5 kanter kobler dokumentrekken (n1 -> n2 -> n3 -> n4 -> n5 -> n6) med rette piler
- Alle noder bruker `allHandles: true` for fri kanttilkobling

**Endret: `Frontend/src/App.jsx`**
- Importstier oppdatert fra `9001-Undersider/*.jsx` til `9001-Undersider/Dynamiske/*.jsx`
- Ny rute lagt til for `StrategiOgStyring`

**Endret: `Frontend/src/components/FlowCRTNode.jsx`**
- Første endring denne dagen: oppsett for skjuling av handles (fulgt opp i neste commit)

**Statistikk:** 20 filer endret, 172 linjer lagt til, 32 linjer slettet

---

### 2. Rollebasert redigering, flow-persistens og handle-kontroll (`199bf09`)

**Tidspunkt:** 14:57

**Endret: `Frontend/src/layout/useFlowPage.js`**
- Ny parameter `canEdit` (boolean): når `false` settes alle noder til `draggable: false` og `editable: false`
- Ny parameter `pageSlug` (string): slug-identifikator for API-kallet
- `useEffect` ved montering: henter lagret flow fra `GET /flows/{pageSlug}` hvis `pageSlug` er satt
- Ny `saveFlow`-callback: kaller `POST /flows/{pageSlug}` med gjeldende `nodes` og `edges` som JSON
- API-URL konfigureres via miljøvariabelen `VITE_API_BASE_URL` (fallback: `http://localhost:8000`)
- `saveFlow` eksporteres som del av hook-returverdien

**Endret: `Frontend/src/pages/ISO-sider/9001-Undersider/Dynamiske/FlowPageTemplate.jsx`**
- Importerer `useAuth` fra `AuthContext`
- `canEdit` utledes fra innlogget brukers rolle: `FLOW_ADMIN` eller `SYSTEM_ADMIN` gir redigeringstilgang
- Ny prop `pageSlug` videresendes til `useFlowPage`
- "Lagre"-knapp vises kun for brukere med `canEdit = true`
- Verktøylinje (legg til node-knapper) skjules for brukere uten redigeringstilgang

**Endret: `Frontend/src/components/FlowCRTNode.jsx`**
- Skriftstørrelse økt: label `20px -> 24px`, description `18px -> 21px`
- Ny logikk for handle-visning (standard modus, ikke `allHandles`):
  - `data.showHandles === false`: skjuler alle handles helt
  - `data.hideTargetHandle`: skjuler venstre target-handle individuelt
  - `data.hideSourceHandle`: skjuler høyre source-handle individuelt

**Endret: `Frontend/src/pages/ISO-sider/ISO9001.jsx`**
- Node `n1` (Salg og kontrakts-håndtering): fikk `hideTargetHandle: true` - ingen innkommende pil
- Node `n7` (Kunde tilfredshet): fikk `hideSourceHandle: true` - ingen utgående pil
- Beskrivelse på Dokumentstyring-noden forkortet til `'Håndtering av dokumenter/prosedyrer'`

**Statistikk:** 5 filer endret, 74 linjer lagt til, 23 linjer slettet

---

## Oppsummering

- **Totalt 25 filer endret/opprettet**, fordelt på 2 faktiske kode-commits (pluss merge-commits)
- Ny tolagsarkitektur for undersider: `Statiske/` for faste visningssider, `Dynamiske/` for redigerbare
- `StaticFlowPage` som gjenbrukbar komponent for statiske sider
- `FlowPageTemplate` + `useFlowPage` kobler nå rollebasert redigering og API-persistens
- `FlowCRTNode` støtter nå selektiv skjuling av handles per node
- `StrategiOgStyring` er første fullstendige underside med predefinert layout

---

## Arkitekturbeslutninger

### Hvorfor skille mellom Statiske/ og Dynamiske/ undersider?

Noen ISO-prosessområder har en fast, autoritativ struktur som ikke skal endres per bruker eller per økt - disse passer i `Statiske/`. Andre prosessområder er beregnet på å redigeres og tilpasses av administratorer, og hører hjemme i `Dynamiske/`. Ved å separere dem i mapper unngår man forvirring om hvilke sider som støtter redigering, og det gjør det enklere å ta grep som å deaktivere drag-and-drop globalt på statiske sider uten å påvirke de dynamiske.

### Hvorfor `useAuth` i `FlowPageTemplate` fremfor i hver enkelt side?

Alle dynamiske undersider arver fra `FlowPageTemplate`, som er det naturlige stedet å hente brukerens rolle. Dersom rollekravet var implementert i hver enkelt `Anskaffelse.jsx`, `Kunder.jsx` osv., ville man måtte oppdatere 12+ filer for én logikkendring. Sentralisering i template-komponenten sikrer konsekvent oppførsel og lett vedlikehold.

### Hvorfor lagre flow via `useFlowPage` og ikke via et separat hook?

`useFlowPage` eier allerede `nodes`, `edges` og `rfInstance`. Det er unaturlig å eksportere state til et eksternt hook bare for å sende det tilbake til API-et. Å legge `saveFlow` direkte i `useFlowPage` holder data nær handlingen, reduserer prop-drilling og er i tråd med prinsippet om at data eies der de forvaltes.

### Diskutert men ikke implementert: page_flows-tabellstruktur

Det ble diskutert en fremtidig `page_flows`-tabell i databasen med kolonnene `page_slug`, `domain_id` og en `flow_data` JSON-kolonne. Dette vil tillate at ulike virksomheter (domains) har sine egne versjoner av hvert flowdiagram, uten at data blandes. `data`-feltet i noder sendes som JSON-objekt (ikke stringified) direkte til API-et.
