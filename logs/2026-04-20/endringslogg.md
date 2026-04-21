# Endringslogg - Søndag 20. april 2026

## Oversikt
En produktiv dag med bidrag fra alle fire gruppemedlemmer. Hans implementerte innsetting av noder mellom eksisterende noder (`insertNodeOnEdge`) og `hasUnsavedChanges`-mekanisme i `useFlowPage.js`. Sondre fikset `list_user_access()` i backend og migrerte hele autentiseringssystemet fra session-basert til JWT. Henrik la til et adminpanel med React Admin. Felix opprettet en ny database-migrasjon.

---

## Commits (kronologisk)

### 1. Implementert list_user_access i backend (`b45bed77`)

**Tidspunkt:** 10:24
**Forfatter:** Sondre

**Endret i `API/src/repositories/users.py`:**
- Implementert `list_user_access(user_id)` som tidligere var en tom funksjon med bare `pass`
- SQL-spørringen joiner `user_access_to_domains`, `domains` og `authorization_levels` for å hente `domain_id`, `company_name` og `role` per domene brukeren har tilgang til
- Returnerer en liste med dictionaries via `row._mapping`

### 2. Innsetting av node mellom eksisterende noder og hasUnsavedChanges (`0bbc3444`)

**Tidspunkt:** 13:27
**Forfatter:** Hans

**Endret i `Frontend/src/auth/useFlowPage.js`:**
- Lagt til `insertNodeOnEdge`-funksjon: velg en edge, klikk "Sett inn" — en ny node plasseres i midtpunktet mellom source og target, og target forskyves med halve avstanden slik at gapet bevares
- Handles på den nye noden bestemmes automatisk basert på den originale edgens `sourceHandle` via et `oppositeHandle`-oppslag
- Den gamle edgen erstattes med to nye: source→ny node og ny node→target
- Lagt til `selectedEdge`-state og `onEdgeClick`/`onPaneClick`-callbacks for edge-seleksjon
- Lagt til `hasUnsavedChanges`-state som settes til `true` ved alle meningsfulle endringer (filtrerer ut `select` og `dimensions` fra `onNodesChange`) og tilbakestilles etter lagring

**Endret i `Frontend/src/pages/9001-Undersider/Dynamiske/FlowPageTemplate.jsx`:**
- Koblet opp `onEdgeClick`, `onPaneClick`, `selectedEdge`, `hasUnsavedChanges` og `insertNodeOnEdge` fra `useFlowPage`
- "Sett inn"-knapp i toolbar som er aktiv kun når en edge er valgt
- Lagre-knapp deaktiveres (`disabled`) når `hasUnsavedChanges` er `false`

### 3. Installer react-helmet-async (`80244799`)

**Tidspunkt:** 13:29
**Forfatter:** Hans

**Endret i `Frontend/package-lock.json`:**
- Ryddet opp overflødige entries i lockfilen (11 slettede linjer)

### 4. Merge pull request #18 (`641717cc`)

**Tidspunkt:** 13:22

Merge av main-branchen inn i prosjektet.

### 5. Slug-endringer i API (`719b21fb`)

**Tidspunkt:** 14:00
**Forfatter:** Sondre

**Endret i `API/src/repositories/users.py`:**
- Forenkling av eksisterende kode

**Endret i `API/src/routers/flows.py`:**
- Justerte flows-rutene for å samsvare med frontend sin bruk av `pageSlug`

### 6. Ny database-migrasjon (`0e11e64e`)

**Tidspunkt:** 14:30
**Forfatter:** Felix

**Ny fil: `Database/migration/V4_1_3__small_append_to_template_db.sql`:**
- Opprettet migrasjonsfil (tom — innhold skal fylles inn)

### 7. React Admin lagt til prosjektet (`70f3a552`)

**Tidspunkt:** 14:35
**Forfatter:** Henrik

**Endret i `Frontend/package.json` og `Frontend/package-lock.json`:**
- La til `react-admin` og `ra-data-fakerest` som avhengigheter
- Stor endring i lockfilen (~1400 nye linjer) grunnet react-admin sitt avhengighetstre

### 8. Adminpanel implementert (`42ba93c1`)

**Tidspunkt:** 14:36
**Forfatter:** Henrik

**Ny fil: `Frontend/src/admin/AdminPanel.jsx` (91 linjer):**
- Komplett adminpanel bygget med `react-admin` og `fakeRestProvider`
- CRUD-operasjoner for brukere (UserList, UserCreate, UserEdit) med felt for name, email og rolle (dropdown med alle 5 FlowCRT-roller)
- CRUD-operasjoner for produkter (ProductList, ProductCreate, ProductEdit) med felt for name, description og price
- Kjører med eget `BrowserRouter` med `basename="/admin"`

**Ny fil: `Frontend/src/admin/data.js` (14 linjer):**
- Hardkodet testdata med 3 brukere og 3 produkter (ISO 9001, 14001, 27001) for fakeRestProvider

**Endret i `Frontend/src/App.jsx`:**
- La til route-sjekk: dersom URL starter med `/admin`, rendres `AdminPanel` i stedet for hovedapplikasjonen
- Import av `AdminPanel`-komponenten

### 9. Fikset tom migrasjonsfil (`048984b8`)

**Tidspunkt:** 15:15
**Forfatter:** Sondre

Fix av tom fil som ble pushet ved en feil.

### 10. Migrering fra session til JWT (`8682af77`)

**Tidspunkt:** 18:15
**Forfatter:** Sondre

**Slettet `API/src/repositories/sessions.py` (54 linjer):**
- Hele session-repositoryet fjernet: `create_session`, `get_session_with_user` og `delete_session`
- Ingen behov for database-baserte sesjoner når JWT brukes

**Endret i `API/.env`:**
- Fjernet `SESSION_EXPIRE_MINUTES`
- Lagt til `JWT_SECRET` (256-bit hex-nøkkel), `JWT_ALGORITHM=HS256` og `JWT_EXPIRE_MINUTES=60`

**Endret i `API/requirements.txt`:**
- Lagt til `python-jose[cryptography]==3.3.0` for JWT-signering og verifisering

**Endret i `API/src/app_config.py`:**
- Fjernet `SESSION_EXPIRE_MINUTES`
- Lagt til `JWT_SECRET`, `JWT_ALGORITHM` og `JWT_EXPIRE_MINUTES` med RuntimeError dersom `JWT_SECRET` mangler

**Endret i `API/src/deps/auth.py`:**
- `get_current_user()` leser nå `access_token`-cookie i stedet for `session_id`
- Dekoder JWT med `jose.jwt.decode()` og henter `sub` (user_id), `email` og `access` direkte fra payload
- Fjernet all database-oppslag for sesjonsvalidering
- Import av `sessions`-repository fjernet, erstattet med `jose` og `app_config`

**Endret i `API/src/routers/auth.py`:**
- Login-endepunktet genererer nå et JWT-token med `jwt.encode()` og setter det som `access_token`-cookie
- Logout sletter `access_token`-cookien i stedet for å slette en database-sesjon

---

## Oppsummering
- **~12 filer endret** på tvers av frontend, backend og database
- Hele autentiseringssystemet migrert fra session-basert (database) til JWT (stateless)
- Ny frontend-funksjonalitet: insertNodeOnEdge og hasUnsavedChanges for bedre flyt-redigering
- Adminpanel med React Admin for bruker- og produktadministrasjon (foreløpig med fake data)
- `list_user_access()` implementert i backend — kritisk for rollebasert tilgangskontroll
- Gruppesamarbeid: alle 4 medlemmer bidro med commits denne dagen

---

## Begrunnelse

### Hvorfor migrere fra session til JWT?

Session-basert autentisering krevde et databaseoppslag ved hver forespørsel for å validere sesjonen. Med JWT er tokenet selvbeskrivende — brukerdata (id, email, roller) ligger innebygd i tokenet og verifiseres kryptografisk uten database. Dette reduserer latens og fjerner avhengigheten til `user_sessions`-tabellen (som hadde en stavefeil: `user_seassions`). JWT er også bedre egnet for en eventuell fremtidig overgang til mikrotjenester der flere tjenester trenger å verifisere autentisering uavhengig.

### Hvorfor insertNodeOnEdge med midtpunktsplassering?

Alternativet hadde vært å plassere den nye noden på en fast posisjon og la brukeren dra den manuelt. Ved å beregne midtpunktet mellom source og target, og forskyve target med halve avstanden, bevares den visuelle flyten i diagrammet automatisk. Brukeren slipper manuell justering og kan umiddelbart begynne å redigere den nye noden. Denne tilnærmingen er inspirert av hvordan Figma og lignende verktøy håndterer innsetting mellom elementer.

### Hvorfor adminpanel med fakeRestProvider?

Backend-endepunktene for bruker- og produktadministrasjon er ikke ferdigstilt. Ved å bruke `ra-data-fakerest` fra React Admin kan Henrik utvikle og teste adminpanelet uavhengig av backend. Når API-et er klart, er det kun dataprovideren som trenger å byttes ut — all UI-kode forblir uendret.

---

## Ikke-koderelatert arbeid

I tillegg til kodeendringene ble det brukt tid på:

1. **Koordinering mellom gruppemedlemmer** — flere parallelle branches ble merget inn i main, noe som krevde kommunikasjon om konflikter og rekkefølge
2. **Testing av JWT-migrasjonen** — manuell verifisering av at innlogging, tilgangskontroll og utlogging fungerer med det nye token-systemet
3. **Planlegging av adminpanelets videre utvikling** — diskuterte hvilke admin-funksjoner som trengs og hvordan de skal kobles mot ekte API
