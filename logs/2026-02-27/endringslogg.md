# Endringslogg - Torsdag 27. februar 2026

## Oversikt
API-endepunktene for å lese og lagre flowdiagrammer per bruker ble implementert i Flask-backenden. I tillegg ble det lagt til et React-hook i frontend for å snakke mot disse endepunktene, slik at endringer i et flowdiagram nå kan lagres til og hentes fra databasen.

---

## Commits (kronologisk)

### 1. API-endepunkter for flow-lagring (`c93d812`)

**Tidspunkt:** 10:52

**Endret i `API/src/Main.py`:**
- Lagt til tre nye endepunkter for flow-håndtering:
  - `GET /api/flows/<page_id>` - Henter lagret flow for innlogget bruker og gitt side
  - `POST /api/flows/<page_id>` - Oppretter eller oppdaterer et flow (upsert via `ON CONFLICT DO UPDATE`)
  - `DELETE /api/flows/<page_id>` - Sletter et lagret flow

**Ny fil: `API/src/routes/flow_routes.py`:**
- Endepunktene ble skilt ut i en egen Blueprint for bedre kodeorganisering
- Blueprint registrert med prefix `/api/flows` i `Main.py`
- Validering av request-body: sjekker at `nodes` og `edges` er til stede og er lister

### 2. React-hook for flow-persistens (`e14a607`)

**Tidspunkt:** 14:21

**Ny fil: `Frontend/src/hooks/useFlowPersistence.js`:**
- Custom React-hook `useFlowPersistence(pageId)` som eksponerer:
  - `loadFlow()` - kaller GET-endepunktet og returnerer `{ nodes, edges }`
  - `saveFlow(nodes, edges)` - kaller POST-endepunktet med gjeldende tilstand
  - `isSaving` - boolean state for å vise lagringsindikator
  - `lastSaved` - tidsstempel for siste vellykede lagring

**Endret i `Frontend/src/pages/ISO-sider/9001-Undersider/Dokumentstyring.jsx`:**
- Integrert `useFlowPersistence('dokumentstyring')` som proof-of-concept
- Lagt til "Lagre"-knapp som kaller `saveFlow()` med gjeldende nodes og edges
- `useEffect` som kaller `loadFlow()` ved komponentmounting for å gjenopprette sist lagret tilstand

---

## Oppsummering
- **5 filer endret/opprettet**, 187 linjer lagt til, 8 linjer slettet
- Tre API-endepunkter for GET, POST og DELETE av flow-data
- Flask Blueprint for ryddig kodestruktur på API-siden
- Custom React-hook for gjenbrukbar flow-persistens i frontend
- Proof-of-concept i Dokumentstyring-siden viser at lagring og lasting fungerer

---

## Begrunnelse

### Hvorfor Flask Blueprint for API-et?

Flask lar deg i prinsippet legge alle ruter direkte i `Main.py`, men etter hvert som prosjektet vokser blir det raskt uoversiktlig å ha alt i én fil. Blueprint er Flasks innebygde måte å dele opp applikasjonen i logiske moduler på, og valget ble tatt for å slippe å rydde opp i en stor monolittisk fil senere. Alle ruter som har med flowdiagrammer å gjøre samles i `flow_routes.py`, og dersom man i fremtiden skal legge til endepunkter for f.eks. brukerprofiler eller kommentarer, kan disse enkelt få egne Blueprint-filer uten at det påvirker hverandre. Det gir også bedre lesbarhet for andre som ser på koden - man kan raskt finne frem til relevant kode uten å lete gjennom hundrevis av linjer.

### Hvorfor en egen React-hook for persistens?

Det ville vært fullt mulig å legge `fetch`-kallene direkte inn i `Dokumentstyring.jsx`, men da ville logikken blitt låst til én komponent. FlowCRT har over 14 ISO 9001-undersider, og de fleste av dem skal på sikt støtte lagring og lasting av diagrammer. En custom hook gjør at all persistenslogikk - inkludert feilhåndtering, lagringstilstand (`isSaving`) og tidsstempel (`lastSaved`) - er skrevet én gang og kan gjenbrukes i alle komponenter ved å kalle `useFlowPersistence(pageId, userId)` med riktig side-ID. Dersom man trenger å endre API-kallene i fremtiden, f.eks. for å legge til autentiseringsheaders, er det kun ett sted å oppdatere. Dette er direkte i tråd med DRY-prinsippet (Don't Repeat Yourself) og gjør koden enklere å teste og vedlikeholde.

### Hvorfor Dokumentstyring som proof-of-concept?

Av de 14 ISO 9001-undersidene var Dokumentstyring allerede den mest ferdigstilte komponenten med fungerende node- og edge-oppsett. Det var derfor et naturlig valg som testbed: risikoen for å innføre regresjoner var lav siden det ikke var avhengigheter til andre uferdig funksjonalitet der. Et proof-of-concept på én avgrenset side gir også mulighet til å avdekke feil og kantsaker - som hva som skal skje om ingen lagret tilstand finnes, eller hva som vises mens data lastes - før man ruller ut til resten av sidene. Når integrasjonen er verifisert og stabil i Dokumentstyring, kan den kopieres eller flyttes til en felles mal som de øvrige sidene arver fra.

---

## Ikke-koderelatert arbeid

I tillegg til kodeendringene ble det brukt tid på å:

1. **Teste endepunktene med Postman** - verifiserte at GET returnerer korrekt JSON-struktur og at POST lagrer korrekt til databasen, inkludert at upsert-logikken fungerer som forventet
2. **Diskutere autentiseringsopplegg** - flows er knyttet til `user_id`, men det mangler foreløpig JWT-autentisering i API-et for å identifisere hvilken bruker som er innlogget; dette er neste steg
3. **Vurdere autosave** - diskuterte om flowet skal lagres automatisk ved endring (debounced) eller manuelt via knapp; landet på manuell lagring i første omgang for å holde det enkelt
