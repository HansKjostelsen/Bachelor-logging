# Endringslogg - Torsdag 5. mars 2026

## Oversikt
Frontend-applikasjonen ble gjort fullstendig uavhengig av backend ved å erstatte API-kall med localStorage-basert datalagring. `AuthContext.jsx` ble omskrevet til å håndtere autentisering lokalt i nettleseren, og `useFlowPage.js` ble oppdatert til å lagre og hente flowdiagrammer fra localStorage i stedet for API.

---

## Commits (kronologisk)

### 1. AuthContext omskrevet til demo-modus (`a3f1c28`)

**Tidspunkt:** 09:34

**Endret i `Frontend/src/context/AuthContext.jsx`:**
- Fjernet alle `fetch`-kall mot `/auth/login` og `/auth/register` API-endepunkter
- Lagt til `SEED_USERS`-konstant med hardkodede testbrukere (én vanlig bruker og én admin) slik at man kan logge inn uten en kjørende backend
- Implementert `login()`-funksjon som sjekker oppgitt e-post og passord mot `SEED_USERS` og mot brukere lagret i localStorage under nøkkelen `flowcrt.demo.users`
- Sesjonsinformasjon lagres nå i localStorage under nøkkelen `flowcrt.demo.session` ved vellykket innlogging, og slettes ved utlogging
- `useAuth()`-hooken henter nå sesjonsdata fra localStorage ved oppstart, slik at innlogget tilstand overlever en sideopplasting

**Ny fil: ingen** — alle endringer i eksisterende fil

### 2. useFlowPage oppdatert til localStorage-lagring (`b8e2d51`)

**Tidspunkt:** 14:07

**Endret i `Frontend/src/hooks/useFlowPage.js`:**
- Fjernet `fetch`-kall mot `GET /flows/{pageSlug}` og `POST /flows/{pageSlug}`
- Flows lagres nå i localStorage med en bruker- og side-spesifikk nøkkel: `flowcrt.flows.{email}.{slug}`
- Ved lasting hentes verdien fra localStorage og parses som JSON; er nøkkelen tom, returneres standardnodene for siden
- Ved lagring serialiseres noder og kanter til JSON og skrives til localStorage
- Ulike innloggede brukere vil nå se sine egne flows, da nøkkelen er knyttet til brukerens e-postadresse

---

## Oppsummering
- **2 filer endret**, ca. 90 linjer lagt til, 55 linjer slettet
- Autentisering og flow-lagring fungerer nå uten en kjørende backend
- Seed-brukere gjør det mulig å demonstrere applikasjonen i isolert frontend-miljø
- Per-bruker flows sikrer at ulike brukere ikke overskriver hverandres diagrammer

---

## Begrunnelse

### Hvorfor demo-modus uten backend?

Prosjektet er på et stadium der backend og frontend utvikles parallelt, men ikke alltid er synkronisert. For å kunne demonstrere og teste frontend-funksjonalitet uten å være avhengig av at Docker Compose-stacken kjører, ble det besluttet å implementere en fullstendig localStorage-basert modus. Dette gjør det mulig å kjøre kun Vite-dev-serveren og likevel ha et fungerende autentiseringsopplegg og persistente flowdiagrammer. Tilnærmingen er midlertidig og designet slik at overgangen til ekte API-kall kan gjøres ved å bytte ut localStorage-operasjonene med `fetch`-kall, uten at resten av komponenttreet trenger å endres.

### Hvorfor per-bruker localStorage-nøkler?

Dersom alle flows ble lagret under én felles nøkkel, ville en bruker som logger ut og en annen logger inn på samme maskin, se den forrige brukerens diagrammer. Ved å inkludere e-postadressen i nøkkelstrengen (`flowcrt.flows.{email}.{slug}`) isoleres dataen per bruker på en enkel måte. Dette speiler også den tenkte backend-oppførselen, der flows er knyttet til `user_id` i databasen, og gjør det lettere å resonnere rundt tilgangskontroll i en fremtidig API-integrasjon.

---

## Ikke-koderelatert arbeid

I tillegg til kodeendringene ble det brukt tid på å:

1. **Verifisere demo-flyten manuelt** - logget inn som seed-bruker, redigerte et flowdiagram, lastet inn siden på nytt og bekreftet at diagrammet ble gjenopprettet fra localStorage
2. **Dokumentere seed-brukernes legitimasjon** - lagt til kommentar i koden med brukernavn og passord for testbrukerne, slik at andre i gruppen kan bruke dem under utvikling
3. **Vurdere sikkerhet** - diskuterte at passord i kildekoden er akseptabelt i demo-sammenheng, men at dette aldri skal gjøres i produksjon; seed-brukere skal fjernes før eventuell deploy
