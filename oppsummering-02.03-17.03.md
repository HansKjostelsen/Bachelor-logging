# Oppsummering: 25. februar – 17. mars 2026

Dette dokumentet oppsummerer arbeidet som ble gjort i FlowCRT-prosjektet i perioden fra 25. februar til 17. mars 2026. Det er ment som et teknisk grunnlag for bachelor-rapporten og dekker implementasjon, begrunnelser for arkitekturvalg, React- og React Flow-standarder som ble fulgt, samt tekniske utfordringer og løsninger.

---

## 1. Hva som ble implementert i perioden

### 25. februar – Diagnose og håndtering av CSS-feil i React Flow

Den første faktiske arbeidsøkten i perioden ble brukt på å identifisere og rette en visuell feil med React Flow-handles (tilkoblingspunktene på nodene). En enkelt CSS-regel, `top: 51px !important` under `.react-flow__handle`, tvang alle handles til én fast posisjon uavhengig av hvilken side av noden de befant seg på. Feilen ble fikset ved å fjerne den globale regelen og erstatte den med fire retningsspesifikke CSS-klasser (`.react-flow__handle-top`, `...-bottom`, `...-left`, `...-right`). Venstre- og høyre-handles fikk `top: 50%; transform: translateY(-50%)` for alltid å sitte vertikalt sentrert på noden, uavhengig av nodehøyde. Samtidig ble handle-logikken i `FlowCRTNode.jsx` forenklet fra to separate betingede blokker til ett ternary-uttrykk.

### 26. februar – Databasestruktur for flow-lagring

Tabellen `user_flows` ble opprettet via Flyway-migrasjon (`V4__create_user_flows_table.sql`). Tabellen lagrer `nodes` og `edges` som JSONB-kolonner, har fremmednøkkelrelasjon mot `users`, og er pålagt et unikt constraint på `(user_id, page_id)` slik at én bruker kun kan ha ett lagret diagram per ISO-side. En PostgreSQL-trigger (`V5__add_flow_updated_at_trigger.sql`) sørger for at `updated_at` oppdateres automatisk ved enhver endring. Python-klassen `UserFlow` ble opprettet som datamodell med metoder for serialisering (`to_dict()`) og deserialisering fra databaserader (`from_db_row()`). Hjelpefunksjoner `execute_query()` og `execute_write()` ble lagt til i `database.py` for å kapsle inn tilkoblings-pooling og feilhåndtering.

### 27. februar – API-endepunkter og React-hook for persistens

Flask Blueprint `flow_routes.py` ble opprettet med tre endepunkter under `/api/flows`:
- `GET /api/flows/<page_id>` – henter lagret flow for bruker og side
- `POST /api/flows/<page_id>` – oppretter eller oppdaterer via upsert (`ON CONFLICT DO UPDATE`)
- `DELETE /api/flows/<page_id>` – sletter lagret flow

Custom React-hooken `useFlowPersistence(pageId, userId)` ble implementert i frontend med funksjonene `loadFlow()`, `saveFlow()`, og tilstandsvariablene `isSaving` og `lastSaved`. Hooken ble integrert som proof-of-concept i `Dokumentstyring.jsx`, som da fikk en "Lagre diagram"-knapp og automatisk innlasting ved montering.

### 4. mars – To-lagsarkitektur og rollebasert redigering

Dette var den mest strukturelt betydningsfulle dagen i perioden. Tre ting skjedde parallelt:

**Reorganisering av mappestruktur:** Alle eksisterende ISO 9001-undersider ble flyttet fra `9001-Undersider/` til en ny undermappe `9001-Undersider/Dynamiske/`. En tilsvarende mappe `9001-Undersider/Statiske/` ble opprettet. `useFlowPage.js` ble flyttet fra `9001-Undersider/` til `Frontend/src/layout/`.

**Ny komponent `StaticFlowPage.jsx`:** En gjenbrukbar komponent for ISO-sider med faste, forhåndsdefinerte noder og kanter. Komponenten tar inn `pageTitle`, `initialNodes` og `initialEdges` som props og viser diagrammet uten noen redigeringsfunksjonalitet.

**Rollebasert redigering i `FlowPageTemplate.jsx`:** Komponenten ble koblet mot `AuthContext` slik at `canEdit` utledes fra innlogget brukers rolle – kun `FLOW_ADMIN` og `SYSTEM_ADMIN` kan redigere. `canEdit`-flagget sendes ned til `useFlowPage`, som setter `draggable: false` og `editable: false` på alle noder for brukere uten redigeringstilgang. Lagreknappen og verktøylinjen skjules for disse brukerne.

**API-persistens i `useFlowPage.js`:** Hooken fikk to nye parametere (`canEdit` og `pageSlug`) og implementerte `GET /flows/{pageSlug}` ved montering og `POST /flows/{pageSlug}` via `saveFlow()`-callback. `credentials: 'include'` sikrer at sesjonskookien sendes med hvert kall.

**Selektiv handle-kontroll i `FlowCRTNode.jsx`:** Tre nivåer av handle-synlighet ble implementert for standardmodus: `showHandles === false` (ingen handles), `hideTargetHandle` (skjuler innkommende), og `hideSourceHandle` (skjuler utgående). Brukes på ISO9001-hovednoden der første node ikke skal ha innkommende pil og siste node ikke skal ha utgående.

**Ny underside `StrategiOgStyring.jsx`:** Første dynamiske underside med forhåndsdefinert node- og kant-layout – 11 noder og 5 kanter.

### 5. mars – Demo-modus: LocalStorage erstatter backend

For å gjøre frontend fullstendig uavhengig av backend ble `AuthContext.jsx` omskrevet og `useFlowPage.js` oppdatert:

**`AuthContext.jsx`** fikk hardkodede `SEED_USERS` (én admin og én vanlig bruker) og en liste med dynamiske brukere under nøkkelen `flowcrt.demo.users` i localStorage. Sesjonsinformasjon lagres i `flowcrt.demo.session` ved innlogging og slettes ved utlogging. `useState` initialiseres med en funksjon som leser fra localStorage, slik at innlogget tilstand overlever sideopplasting.

**`useFlowPage.js`** erstattet `fetch`-kall med localStorage-operasjoner. Flows lagres under nøkkelen `flowcrt.flows.{email}.{slug}`, noe som isolerer data per bruker og per side.

### 17. mars – Fullstendig demo-flyt: Registrering, kjøp og lagringsstatus

**Registrering via `register()`:** Ny funksjon i `AuthContext` som oppretter bruker i localStorage under `flowcrt.demo.users` med rollen `VIEWER`. Kaster feil dersom e-posten allerede er registrert. `Register.jsx` ble oppdatert til å bruke `useAuth().register()` i stedet for direkte `fetch`-kall, og kaller automatisk `login()` etter vellykket registrering.

**Rolleoppgradering via `buyStandard()`:** Funksjon i `AuthContext` som oppgraderer innlogget brukers rolle fra `VIEWER` til `FLOW_ADMIN`. Rolleendringen persisteres i begge localStorage-nøklene (`flowcrt.demo.users` og `flowcrt.demo.session`) slik at endringen er synlig umiddelbart og overlever utlogging.

**`hasUnsavedChanges` i `useFlowPage.js`:** Boolean-state som settes til `true` ved enhver node- eller kant-endring og tilbakestilles til `false` etter lagring eller innlasting. Eksponeres fra hooken slik at komponenter kan deaktivere eller style lagreknappen basert på tilstanden.

---

## 2. React-standarder som ble fulgt

### Custom hooks for logikk-separasjon

Kjernen i React-arkitekturen er at forretningslogikk og tilstandshåndtering er skilt ut i egne hooks: `useFlowPage` eier `nodes`, `edges`, `hasUnsavedChanges`, `saveFlow`, og `loadFlow`. `FlowPageTemplate` er en ren presentasjonskomponent som delegerer til hooken og `AuthContext` for å avgjøre hva som skal vises.

Dette er korrekt bruk av separasjonen mellom _stateful logic_ og _UI_ i React. Begrunnelsen er konkret: dersom `saveFlow`-logikken hadde ligget direkte i `FlowPageTemplate`, ville det vært umulig å teste den isolert, og den måtte blitt kopiert til alle 14 ISO-undersider. Ved å samle logikken i `useFlowPage` er det kun én fil å oppdatere – for eksempel ved overgangen fra localStorage til ekte API-kall.

### useCallback for stabile funksjonsreferanser

`onNodesChange`, `onEdgesChange`, og `onConnect` i `useFlowPage` er pakket inn i `useCallback` med relevante avhengigheter. Dette hindrer at React Flow re-renderer unødvendig fordi callback-referansen endres ved hver render. Uten `useCallback` ville React Flow motta nye funksjonsreferanser ved hvert render og kunne trigge unødvendige interne effekter.

### useState med initialisatorfunksjon

`AuthContext` bruker `useState(() => { ... })` i stedet for `useState(verdi)` for å lese fra localStorage. Initialisatorfunksjonen kjøres kun én gang ved montering, i motsetning til et direkte `localStorage.getItem()`-kall som ville evalueres ved hvert render. Dette er den React-anbefalte måten å initialisere state fra en ekstern kilde som kan feile (f.eks. korrupt JSON).

### Context API for global tilgangsstyring

`AuthContext` bruker Reacts innebygde Context API for å gjøre brukerinformasjon og autentiseringsfunksjoner tilgjengelig i hele komponenttreet uten prop-drilling. `FlowPageTemplate` leser `user.role` via `useAuth()` – den trenger ikke å motta `user` som prop fra en forelder. Dette er standard bruksmønster for global tilstand som ikke endres hyppig.

### Prop-grensesnitt holdt minimalt

`StaticFlowPage` tar inn kun `pageTitle`, `initialNodes`, og `initialEdges`. Dette er bevisst minimal API-design: en komponent skal ha akkurat det antallet props som trengs for å gjøre jobben sin, ikke mer. Et bredt prop-grensesnitt er et tegn på at komponenten gjør for mye.

### Separation of concerns mellom Statiske og Dynamiske undersider

ISO 9001-undersider er delt i to mapper. `Dynamiske/` inneholder sider koblet mot `useFlowPage`, `AuthContext`, og lagrings-API-et. `Statiske/` inneholder sider som kun viser et fast diagram. De to mappene følger Single Responsibility-prinsippet: dynamiske komponenter har ett ansvar (redigerbart diagram med persistens), statiske har ett annet (visning av fastsatt prosessflyt).

---

## 3. React Flow-standarder som ble fulgt

### nodeTypes definert utenfor komponenten

`nodeTypes`-objektet er definert som en konstant utenfor komponentfunksjonen i alle flow-komponenter:

```javascript
const nodeTypes = {
  flowcrt: FlowCRTNode,
  label: LabelNode,
};
```

Dette er et krav fra React Flow. Dersom `nodeTypes` ble definert inne i komponenten, ville objektreferansen endres ved hvert render og React Flow ville avmontere og gjenopprette alle noder. Dette er en vanlig feil som gir både ytelsesforringelse og visuell flimring.

### onNodesChange og onEdgesChange via applyNodeChanges / applyEdgeChanges

React Flow forventer at eksterne tilstandsoppdateringer håndteres via de offisielle hjelpefunksjonene:

```javascript
const onNodesChange = useCallback(
  (changes) => setNodes((nds) => applyNodeChanges(changes, nds)),
  [],
);
```

Dette er React Flows anbefalte mønster for _controlled flow_. Alternativet er _uncontrolled flow_ der React Flow håndterer tilstanden internt, men det gir ikke mulighet til å lytte på eller modifisere endringer – noe som kreves for `hasUnsavedChanges`-mekanismen.

### fitView og responsiv resize-lytter

Alle flow-komponenter bruker `fitView` som prop på `<ReactFlow>`-elementet og registrerer en `resize`-lytter via `useEffect`:

```javascript
useEffect(() => {
  if (!rfInstance) return;
  const handleResize = () => rfInstance.fitView({ padding: 0.2 });
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, [rfInstance]);
```

`fitView` som React Flow-prop sørger for at diagrammet zoomes til å passe inn i visningsvinduet ved første render. Resize-lytteren sikrer at dette opprettholdes når vindusbredden endres – uten den ville diagrammet flyte ut av canvas ved endring av nettleserstørrelse. `cleanup`-funksjonen i `useEffect` fjerner lytteren ved avmontering for å unngå minnelekkasjer.

### Handles og custom node-design

`FlowCRTNode` implementerer React Flows custom node-API korrekt. Noden eksporterer sine handles som `<Handle>`-komponenter med eksplisitte `id`-attributter (`"left-target"`, `"right-source"` osv.). Kanter i `StrategiOgStyring` refererer til disse eksplisitt via `sourceHandle` og `targetHandle`:

```javascript
{ id: 'n1-n2', source: 'n1', target: 'n2',
  sourceHandle: 'right-source', targetHandle: 'left-target', ... }
```

Dette er React Flows anbefalte mønster for noder med flere handles. Uten eksplisitte handle-IDer vil React Flow velge tilkoblingspunkt automatisk, noe som kan gi uønsket visuell oppførsel.

`allHandles`-modus i `FlowCRTNode` viser 8 handles (topp, bunn, venstre, høyre, og de fire hjørnene) og brukes på undersider der administratorer trenger full frihet til å koble noder i alle retninger. Standard-modus viser kun venstre (target) og høyre (source), som passer til lineære prosessflyter.

### MarkerType for pilspiss

Kanter bruker `markerEnd: { type: MarkerType.ArrowClosed, color: 'black' }` for pilspisser. `MarkerType`-enum fra `@xyflow/react` er den anbefalte måten å deklarere piltyper på, fremfor hardkodede strenger, da den sikrer type-sikkerhet og kompatibilitet på tvers av React Flow-versjoner.

### minZoom og proOptions

Alle flow-komponenter setter `minZoom={0.3}` for å tillate mer utzoom enn standardverdien, og `proOptions={{ hideAttribution: true }}` for å skjule React Flow-logoen. Dette er standard konfigurasjon for produksjonslignende grensesnitt.

---

## 4. Begrunnelse for arkitekturvalg

### To-lagsarkitektur: Statiske/ og Dynamiske/

Skillet mellom statiske og dynamiske undersider er ikke kun organisatorisk – det reflekterer to fundamentalt forskjellige bruksmønstre. Statiske sider viser en autoritativ, forhåndsdefinert prosessflyt som representerer ISO-standarden slik den er. Dynamiske sider er arbeidsflater der administratorer tilpasser diagrammet til sin virksomhet. Ved å plassere dem i separate mapper er det umiddelbart tydelig for en utvikler hvilken kategori en ny side tilhører, og hvilken template den skal arve fra. Det forhindrer at man ved en feil legger redigeringsfunksjonalitet inn i en side som skal være låst.

### LocalStorage for demo-modus

Prosjektet befinner seg i en fase der frontend og backend utvikles parallelt, men ikke alltid er synkronisert. Å implementere en fullstendig localStorage-basert autentisering og flow-lagring gjør det mulig å demonstrere hele brukerreisen – registrering, kjøp av tilgang, redigering, lagring, utlogging og gjenopprettelse av data – uten at Docker Compose-stacken er oppe. Dette er verdifullt for presentasjoner, testing og rask iterasjon. Arkitekturen er bevisst utformet slik at overgangen til ekte API-kall kun krever endringer i `AuthContext.jsx` og `useFlowPage.js` – resten av komponenttreet er uberørt.

### Flows lagres per bruker (per-bruker localStorage-nøkler)

Nøkkelformatet `flowcrt.flows.{email}.{slug}` isolerer flows per bruker og per side. Alternativet – en felles nøkkel per side – ville ført til at én brukers endringer overskriver en annens på samme maskin. Per-bruker-isolasjon speiler også den tenkte backend-atferden (`user_id`-kolonne i `user_flows`-tabellen), noe som gjør det enklere å resonnere rundt tilgangskontroll og letter fremtidig migrasjon til ekte API.

### hasUnsavedChanges som separat boolean-state

En alternativ tilnærming ville vært å sammenligne gjeldende `nodes`/`edges` med det som er lagret for å avgjøre om det finnes endringer. Dette ville kreve dyp sammenligning av potensielt store dataobjekter ved hvert render – svært kostbart. En enkel boolean som settes til `true` ved enhver strukturell endring og tilbake til `false` ved lagring er billig, lettvint å resonnere rundt, og gir nøyaktig nok feedback for formålet. Den ene kjente svakheten – at manuell angring ikke nullstiller flagget – er akseptabel for demo-formål.

### Automatisk innlogging etter registrering

Å kalle `login()` direkte etter `register()` i `Register.jsx` eliminerer et ekstra steg for brukeren. Dette er vanlig UX-praksis og tilsvarer det en backend ville gjort ved å returnere en sesjonscookie direkte i register-responsen. Implementasjonen er enkel fordi begge funksjonene er synkrone localStorage-operasjoner.

### Persistering av rolleendring i begge localStorage-nøkler

`buyStandard()` oppdaterer både brukeroppføringen i `flowcrt.demo.users` og den aktive sesjonen i `flowcrt.demo.session`. Uten begge oppdateringene ville enten den nåværende UI-visningen ikke reflektere den nye rollen (om kun brukeroppføringen ble oppdatert), eller neste innlogging ville tilbakestille til `VIEWER` (om kun sesjonen ble oppdatert). Å oppdatere begge sikrer full konsistens.

### JSONB fremfor normaliserte tabeller

React Flow representerer et diagram som to JavaScript-arrays: noder og kanter. Å lagre disse direkte som JSONB-kolonner i databasen er naturlig fordi det speiler dataformatet som allerede eksisterer i applikasjonen. Normalisering til separate tabellrader ville krevd kompleks serialisering og deserialisering, mer kode, og JOIN-operasjoner ved henting – uten at det gir noen reell fordel gitt at hele flowen alltid hentes som én enhet.

### UNIQUE constraint og upsert-mønster

`UNIQUE (user_id, page_id)` i `user_flows`-tabellen gjør at `INSERT ... ON CONFLICT DO UPDATE` håndterer både opprettelse og oppdatering i én databaseoperasjon. Dette forenkler API-endepunktet betraktelig: frontend trenger ikke å skille mellom "opprett" og "oppdater" – den sender alltid `POST`, og databasen håndterer det riktig.

### Flask Blueprint for API-organisering

Flask tillater at alle ruter legges i `Main.py`, men det er ikke skalerbart. Blueprint separerer flow-logikken i en dedikert fil (`flow_routes.py`) og gjør kodebasen lettere å navigere. En utvikler som skal endre flow-endepunktene vet nøyaktig hvilken fil som er relevant. Det åpner også for å legge til nye Blueprint-moduler (f.eks. for brukerprofilhåndtering) uten at de påvirker hverandre.

---

## 5. Tekniske utfordringer og løsninger

### CSS-konflikten med React Flow-handles (25. februar)

**Utfordringen:** Handles (tilkoblingspunktene på nodene) sto på feil posisjon. Venstre- og høyre-handles sto midt inne i noden i stedet for på kantene, noe som gjorde dem ubrukelige for kantopprettelse.

**Årsaken:** En enkelt CSS-regel, `top: 51px !important` under `.react-flow__handle`, overstyrer alle av React Flows egne posisjonsberegninger for handles. `!important` gjør at regelen har høyere spesifisitet enn React Flows interne stiler, uavhengig av hvilken side av noden handelen er ment å representere.

**Løsningen:** Fjernet den globale regelen og erstattet den med fire retningsspesifikke CSS-klasser som React Flow selv tilbyr. For venstre og høyre handles ble `top: 50%; transform: translateY(-50%)` brukt for å sikre vertikal sentrering uavhengig av nodehøyde. Dette er å arbeide _med_ rammeverket, ikke mot det: React Flow har disse klassene nettopp for at utviklere skal kunne tilpasse handles.

**Lærdom:** `!important` i CSS er et tegn på at noe er galt med spesifisitetsstrukturen. I et rammeverk som React Flow, som har et veldefinert CSS-klasseheriarki, er det alltid bedre å bruke rammeverkets egne klasser fremfor å overstyre med `!important`.

### Rollebasert redigering uten prop-drilling (4. mars)

**Utfordringen:** `FlowPageTemplate` er en felles mal for alle 14 dynamiske undersider. Dersom rolleinformasjonen skulle sendes ned som props fra ruteren, måtte alle 14 underside-komponenter oppdateres, og ruterkonfigurasjonen ville blitt komplisert.

**Løsningen:** `FlowPageTemplate` leser brukerens rolle direkte via `useAuth()` fra `AuthContext`. Ingen av undersidene trenger å ta imot `user` som prop. `canEdit`-logikken er samlet på ett sted og dermed lett å endre (f.eks. for å legge til en ny rolle med redigeringstilgang).

### Konsistens mellom UI-tilstand og localStorage ved rolleoppgradering (17. mars)

**Utfordringen:** `buyStandard()` må sørge for at UI-et reflekterer den nye rollen umiddelbart, OG at neste innlogging gir korrekt rolle.

**Løsningen:** Oppdatere begge localStorage-nøklene atomisk i samme funksjon, og deretter kalle `setUser(updatedSession)` for å oppdatere React-tilstanden. Rekkefølgen er viktig: localStorage oppdateres først slik at en eventuell feil i `setUser` ikke etterlater et inkonsistent state.

### Selektiv handle-synlighet (4. mars)

**Utfordringen:** ISO9001-hovednoden (prosessrekken) skal vise handles, men første og siste node i rekken skal ikke ha henholdsvis innkommende og utgående handle – dette er visuelt viktig for å kommunisere start og slutt på prosessen.

**Løsningen:** Tre nivåer av handle-kontroll ble implementert som data-flagg på noden (`showHandles`, `hideTargetHandle`, `hideSourceHandle`). Dette holder kontrollen i data-laget (node-definisjonen) i stedet for i template-koden, noe som er i tråd med React Flow-mønsteret der node-opplevelse styres via `data`-objektet.

### Synkronisering av `hasUnsavedChanges` med load/save (17. mars)

**Utfordringen:** `hasUnsavedChanges` skal settes til `false` ikke bare etter lagring, men også etter innlasting – ellers ville brukeren se "ulagrede endringer" umiddelbart etter at siden lastet inn data fra localStorage.

**Løsningen:** `loadFlow()` kaller `setHasUnsavedChanges(false)` etter at nodes og edges er satt. `saveFlow()` gjør det samme etter vellykket lagring. `handleSetNodes` og `handleSetEdges` (wrapper-funksjonene som pakker inn de originale setterfunksjonene) kaller `setHasUnsavedChanges(true)`. Fordi hooken eksponerer `setNodes` og `setEdges` med de samme navnene som de interne wrapperne, trenger ingen eksisterende komponenter å oppdatere sine variabelnavn.

---

## Tidslinje for perioden

| Dato | Hovedleveranse |
|------|----------------|
| 25.02 | CSS-bugfix for handles, kartlegging av databasebehov |
| 26.02 | `user_flows`-tabell, SQL-trigger, `UserFlow`-modellklasse |
| 27.02 | Flask Blueprint med GET/POST/DELETE, `useFlowPersistence`-hook, proof-of-concept i Dokumentstyring |
| 04.03 | To-lagsarkitektur (Statiske/Dynamiske), `StaticFlowPage`, rollebasert redigering, API-persistens i `useFlowPage`, `StrategiOgStyring` |
| 05.03 | Demo-modus: `AuthContext` og `useFlowPage` omskrevet til localStorage |
| 17.03 | `register()`, `buyStandard()`, `hasUnsavedChanges` – fullstendig demo-flyt |
