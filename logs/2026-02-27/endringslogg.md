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

## Ikke-koderelatert arbeid

I tillegg til kodeendringene ble det brukt tid på å:

1. **Teste endepunktene med Postman** - verifiserte at GET returnerer korrekt JSON-struktur og at POST lagrer korrekt til databasen, inkludert at upsert-logikken fungerer som forventet
2. **Diskutere autentiseringsopplegg** - flows er knyttet til `user_id`, men det mangler foreløpig JWT-autentisering i API-et for å identifisere hvilken bruker som er innlogget; dette er neste steg
3. **Vurdere autosave** - diskuterte om flowet skal lagres automatisk ved endring (debounced) eller manuelt via knapp; landet på manuell lagring i første omgang for å holde det enkelt
