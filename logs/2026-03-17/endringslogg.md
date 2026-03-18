# Endringslogg - Tirsdag 17. mars 2026

## Oversikt
Demo-modusen ble fullstendig ved å implementere brukerregistrering, rolleoppgradering via `buyStandard()` og en `hasUnsavedChanges`-mekanisme i `useFlowPage.js`. Etter disse endringene er hele brukerreisen — fra registrering via kjøp av tilgang til redigering og lagring av flowdiagrammer — operativ uten avhengighet til backend.

---

## Commits (kronologisk)

### 1. Registreringsfunksjon i AuthContext og Register.jsx (`c4a92ef`)

**Tidspunkt:** 10:18

**Endret i `Frontend/src/context/AuthContext.jsx`:**
- Lagt til `register(name, email, password)`-funksjon som oppretter en ny bruker i localStorage under nøkkelen `flowcrt.demo.users`
- Ny bruker får rolle `VIEWER` og et auto-inkrementert ID basert på antallet eksisterende brukere
- Funksjonen kaster en feil dersom e-postadressen allerede er registrert (sjekker mot både seed-brukere og localStorage-brukere)
- `register()` eksponeres via `AuthContext` slik at alle komponenter kan kalle den via `useAuth()`

**Endret i `Frontend/src/pages/Register.jsx`:**
- Oppdatert til å kalle `useAuth().register()` i stedet for direkte `fetch`-kall mot `/auth/register`
- Etter vellykket registrering kalles `login()` automatisk slik at brukeren er innlogget med én gang
- Feilmeldinger fra `register()` (f.eks. "E-post allerede i bruk") vises i skjemaet

### 2. buyStandard for rolleoppgradering (`d07b3a1`)

**Tidspunkt:** 12:45

**Endret i `Frontend/src/context/AuthContext.jsx`:**
- Lagt til `buyStandard()`-funksjon som oppgraderer innlogget brukers rolle fra `VIEWER` til `FLOW_ADMIN`
- Rolleendringen persisteres i begge localStorage-nøklene: brukeroppføringen i `flowcrt.demo.users` og sesjonen i `flowcrt.demo.session`
- `user`-state oppdateres umiddelbart slik at UI-et reflekterer den nye rollen uten sideopplasting
- Seed-brukere kan ikke oppgraderes via `buyStandard()` (de har ikke oppføringer i `flowcrt.demo.users`)

### 3. hasUnsavedChanges i useFlowPage (`f3e8c12`)

**Tidspunkt:** 15:30

**Endret i `Frontend/src/hooks/useFlowPage.js`:**
- Lagt til `hasUnsavedChanges`-state (boolean) som initialiseres til `false`
- Settes til `true` hver gang `setNodes` eller `setEdges` kalles med nye verdier
- Settes tilbake til `false` etter vellykket `saveFlow()`
- Eksponeres fra hooken slik at komponenter kan bruke den til å deaktivere eller style lagreknappen

**Effekt i UI:**
- Lagreknappen i flowsidene er nå deaktivert (`disabled`) når `hasUnsavedChanges` er `false`
- Visuell tilbakemelding: knappen vises med redusert opasitet når det ikke er ulagrede endringer

---

## Oppsummering
- **3 filer endret**, ca. 75 linjer lagt til, 12 linjer slettet
- Fullstendig brukerflyt uten backend: registrer konto → kjøp standard → rediger flow → lagre
- `hasUnsavedChanges` gir bedre UX ved å tydeliggjøre når det finnes endringer som ikke er lagret
- Demo-modusen er nå ansett som ferdig og verifisert manuelt

---

## Begrunnelse

### Hvorfor automatisk innlogging etter registrering?

Det er vanlig UX-praksis at nye brukere ikke tvinges til å logge inn manuelt etter at de nettopp har opprettet en konto — de har allerede oppgitt legitimasjonen sin og forventer å komme rett inn i applikasjonen. Ved å kalle `login()` direkte etter `register()` i `Register.jsx` slipper brukeren dette ekstra steget, og flyten føles mer sømløs. I en produksjonssetting ville dette tilsvare at backenden returnerer en sesjonscookie eller token direkte i register-responsen.

### Hvorfor persistere rolleendring i begge localStorage-nøklene?

`flowcrt.demo.users` inneholder den varige brukeroppføringen, mens `flowcrt.demo.session` representerer den aktive sesjonen. Dersom man kun oppdaterer sesjonen, vil neste innlogging på nytt tildele `VIEWER`-rollen fordi `login()`-funksjonen henter rollen fra brukeroppføringen. Og dersom man kun oppdaterer brukeroppføringen, vil ikke den allerede innloggede brukeren se noen endring i UI-et. Å oppdatere begge sikrer konsistens mellom nåværende sesjon og fremtidige innlogginger.

### Hvorfor hasUnsavedChanges som separat state?

En alternativ tilnærming hadde vært å sammenligne gjeldende nodes/edges med det som er i localStorage ved hver render for å avgjøre om det finnes endringer. Dette ville imidlertid vært kostbart siden det krever dyp sammenligning av potensielt store dataobjekter. En enkel boolean-state som settes til `true` ved enhver nodeendring og tilbake til `false` ved lagring er langt mer effektiv og lettere å resonnere rundt. Ulempen er at den ikke er perfekt nøyaktig — f.eks. vil en endring som angres manuelt likevel holde `hasUnsavedChanges` som `true` — men for demo-formål er dette akseptabelt.

---

## Ikke-koderelatert arbeid

I tillegg til kodeendringene ble det brukt tid på å:

1. **Manuelt verifisere hele demo-flyten** - gjennomgikk stegene: opprett konto → logg inn → kjøp standard → åpne flowside → rediger node → lagre → logg ut → logg inn igjen → verifiser at diagrammet er gjenopprettet
2. **Oppdatere MEMORY.md** - noterte demo-modusen og dens detaljer i agent-minnefilen slik at fremtidige logg-sesjoner har kontekst
3. **Diskutere overgang til ekte API** - planla at når backend er klar, er det kun `AuthContext.jsx` og `useFlowPage.js` som trenger endres for å gå fra localStorage til ekte API-kall
