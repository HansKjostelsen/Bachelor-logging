# Endringslogg - Tirsdag 25. februar 2026

## Oversikt
Dagen gikk med til å debugge og fikse et irriterende problem med React Flow-handles (tilkoblingsprikker) på nodene i FlowCRT. Rotårsaken viste seg å være en enkelt CSS-regel som tvang alle handles til feil posisjon. Etter at feilen var fikset, ble det også brukt tid på å kartlegge hva som kreves for å persistere flowdiagrammer per bruker i databasen.

---

## Commits (kronologisk)

### 1. Merge pull request #11 (`21803dc`)

- Merget PR #11 fra HansKjostelsen/main
- Tidspunkt: 11:29

### 2. Løst handle-problem (`3d941498`)

**Tidspunkt:** 11:56

**Rotårsak identifisert og fikset i `Frontend/src/styles.css`:**
- Regelen `top: 51px !important` under `.react-flow__handle` tvang **alle** handles til å bli plassert 51 piksler ned fra toppen av noden, uavhengig av hvilken side de hørte til
- Dette resulterte i at handles på venstre og høyre side av nodene sto midt inne i noden istedenfor på kantene
- Fikset ved å fjerne den feilaktige regelen og i stedet legge til korrekte CSS-regler per retning:
  - `.react-flow__handle-top`: `top: -5px`
  - `.react-flow__handle-bottom`: `bottom: -5px`
  - `.react-flow__handle-left`: `left: -5px` med `top: 50%` og `transform: translateY(-50%)`
  - `.react-flow__handle-right`: `right: -5px` med `top: 50%` og `transform: translateY(-50%)`

**Forenkling av logikk i `Frontend/src/components/FlowCRTNode.jsx`:**
- Forenklet betingelseslogikken for rendering av handles
- Endret fra to separate `{data.showHandles !== false && data.allHandles && (...)}` og `{data.showHandles !== false && !data.allHandles && (...)}` til ett enkelt ternary-uttrykk: `{data.allHandles ? (...) : (...)}`
- Gjør koden mer lesbar og fjerner redundant `showHandles`-sjekk

---

## Oppsummering
- **2 filer endret**, 22 linjer lagt til, 5 linjer slettet
- Én bugfix-commit med stor visuell effekt
- Handles vises nå korrekt på kantene av nodene

---

## Ikke-koderelatert arbeid

I tillegg til kodeendringene ble det brukt tid på å:

1. **Analysere git-historikk** for å spore rotårsaken til handle-feilen bakover gjennom commits
2. **Kartlegge systemarkitekturen** - gikk gjennom hele FlowCRT-systemet (frontend, backend, database) for å forstå hva som er på plass
3. **Planlegge brukerlagring av flowdiagrammer** - diskuterte hvilke data som må lagres i databasen for å persistere React Flow-diagrammer per bruker:
   - Hva som må lagres: `nodes[]` (id, position, data), `edges[]` (source, target), brukertilknytning, side/kontekst-identifikator, og tidsstempel
   - Eksisterende tabeller i databasen: `users`-tabell er på plass
   - Hva som mangler: ny tabell for `user_flows` eller tilsvarende
   - API-endepunkter for GET/POST/PUT av flow-data per bruker mangler
