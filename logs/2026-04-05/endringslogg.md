# Endringslogg - Søndag 5. april 2026

## Oversikt
Dagen ble brukt på å koble flow-lagringen til localStorage slik at flowdiagrammer persisterer per bruker uten at backend er involvert, og på å innføre `hasUnsavedChanges`-state i `useFlowPage.js` som deaktiverer lagre-knappen inntil faktiske endringer er gjort. Resultatet er at hele lagre/last-syklusen nå fungerer utelukkende i nettleseren.

---

## Commits (kronologisk)

### 1. localStorage-persistens for flows i useFlowPage, per-bruker lagring og hasUnsavedChanges

**Beskrivelse:**
`useFlowPage.js` brukte frem til nå `fetch` mot `GET/POST /flows/{pageSlug}` for å laste og lagre flowdiagrammer. Disse kallene ble erstattet med `localStorage.getItem`/`localStorage.setItem`, og lagringsstrukturen ble gjort bruker-spesifikk: nøkkelformatet `flowcrt.flows.{email}.{slug}` sørger for at to ulike brukere som logger inn på samme maskin ikke deler flowdata. I tillegg ble det innført en `hasUnsavedChanges`-boolean som holder styr på om noder eller kanter er endret siden siste lagring eller innlasting.

**Endringer i `Frontend/src/layout/useFlowPage.js`:**

- Fjernet `fetch`-kall til `GET /flows/{pageSlug}` og `POST /flows/{pageSlug}`
- Importerer `useAuth` for å hente innlogget brukers e-post
- Ny hjelpefunksjon `storageKey(email, slug)` som returnerer nøkkelen `flowcrt.flows.{email}.{slug}`
- `useEffect` ved montering leser fra localStorage med `storageKey` og setter `nodes`/`edges` hvis data finnes
- `saveFlow`-callback skriver til localStorage med `storageKey` og setter `hasUnsavedChanges` til `false`
- `hasUnsavedChanges` settes til `true` i `onNodesChange` og `onEdgesChange` (men ikke ved `select`-endringer, kun ved faktiske posisjon- eller data-endringer)
- `hasUnsavedChanges` eksporteres som del av hook-returverdien

**Statistikk:** 1 fil endret, ~35 linjer lagt til, ~20 linjer slettet

---

## Arkitekturbeslutninger

### Hvorfor per-bruker nøkler i localStorage?

Dersom alle brukere delte samme nøkkel (`flowcrt.flows.{slug}`), ville bruker A se og overskrive bruker B sine endringer når de bruker samme nettleser. Dette er spesielt problematisk i en demo der man ønsker å vise at admin kan ha sine egne layouts. Nøkkelformatet `flowcrt.flows.{email}.{slug}` er enkelt, lesbart i DevTools og kan enkelt tømmes per bruker ved behov.

### Hvorfor ikke lagre automatisk (auto-save)?

En eksplisitt lagre-knapp er pedagogisk riktig: det viser at systemet *kan* lagre, og det gjør det tydelig for veileder at persistens fungerer. Auto-save ville skjult dette. I tillegg er det lettere å demonstrere `hasUnsavedChanges`-mekanismen med en knapp som synlig endrer tilstand.

### Hvorfor filtrere bort select-endringer i hasUnsavedChanges?

React Flow utløser `onNodesChange` selv ved rene select/deselect-operasjoner (klikk på en node). Dersom alle endringer satte `hasUnsavedChanges = true`, ville lagre-knappen aktiveres ved et enkelt klikk uten at noe faktisk er endret. Filtreringen sjekker at endringstypen er noe annet enn `select` før flagget settes.

---

## Neste steg
- Oppdatere `Register.jsx` til å bruke den nye `register()`-funksjonen fra `AuthContext`
- Implementere `buyStandard()` i `AuthContext` som oppgraderer brukerens rolle til `FLOW_ADMIN` og lagrer det i localStorage
