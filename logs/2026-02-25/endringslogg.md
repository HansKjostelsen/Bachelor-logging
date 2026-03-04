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

## Begrunnelse

### Hvorfor ble handle-buggen fikset på denne måten?

Den opprinnelige feilen skyldes at én enkelt CSS-regel med `!important` overstyrer alle React Flows egne posisjonsberegninger for handles. Den naive løsningen ville vært å endre tallet fra `51px` til noe mer passende, men det ville bare flytte problemet - ikke løse det. Rotårsaken er at en global regel aldri kan håndtere fire ulike plasseringer (topp, bunn, venstre, høyre) med én enkelt verdi.

Valget om å fjerne den globale regelen helt og erstatte den med fire retningsspesifikke CSS-klasser er den korrekte tilnærmingen fordi:

- **React Flow har egne klasser per retning** (`.react-flow__handle-top`, `...-left` osv.), som er nettopp designet for denne typen tilpasning. Å bruke disse klassene er å arbeide med rammeverket, ikke mot det.
- **Ingen `!important` er nødvendig** med retningsspesifikke klasser, fordi CSS-spesifisiteten er tilstrekkelig. Å unngå `!important` gjør stilarket mer forutsigbart og lettere å vedlikeholde fremover.
- **`top: 50%; transform: translateY(-50%)`-teknikken** er standardmetoden for vertikal sentrering i CSS og sikrer at venstre- og høyre-handles alltid sitter midt på noden, uavhengig av nodehøyde. Dette gjør løsningen robust mot fremtidige layoutendringer.

Forenkling av handle-logikken i `FlowCRTNode.jsx` til ett ternary-uttrykk ble gjort samtidig fordi det var et naturlig tidspunkt å rydde opp: mens man uansett var inne i filen og forstod handle-logikken, var det lite overhead å gjøre koden enklere. To separate if-blokker med nesten identiske betingelser (`data.showHandles !== false && data.allHandles` vs. `data.showHandles !== false && !data.allHandles`) er et klassisk mønster som kan forenkles til én ternary, noe som reduserer risikoen for at de to blokkene går ut av synk ved fremtidige endringer.

### Hvorfor kartlegge databasestruktur for brukerlagring nå?

Kartleggingen av hva som kreves for å lagre flowdiagrammer per bruker ble gjort på dette tidspunktet av en pragmatisk grunn: når man uansett er dypt inne i React Flow-koden og har god oversikt over nodestrukturen (`id`, `position`, `data`) og kantstrukturen (`source`, `target`), er det naturlig å tenke gjennom hva som faktisk trenger å persisterast. Å utsette denne analysen til senere ville betydd at man måtte sette seg inn i det igjen fra bunnen av.

Analysen avdekket at backend allerede har en `users`-tabell, men mangler tabell og API-endepunkter for flowlagring. Å kjenne til dette gapet tidlig gir et mer realistisk bilde av gjenstående arbeid og gjør det mulig å prioritere riktig i kommende arbeidsøkter.

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
