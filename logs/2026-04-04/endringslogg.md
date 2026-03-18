# Endringslogg - Lørdag 4. april 2026

## Oversikt
Dagen ble brukt på å løsrive frontend-demoen fra backend-avhengigheten ved å omskrive `AuthContext.jsx` til å håndtere autentisering utelukkende via localStorage og forhåndsdefinerte mock-brukere. Dette gjør det mulig å demonstrere hele brukerreisen – registrering, kjøp av standard og rollebasert redigering – uten at backend trenger å kjøre.

---

## Commits (kronologisk)

### 1. Demo-modus i AuthContext: mock-brukere og localStorage-autentisering

**Beskrivelse:**
`AuthContext.jsx` ble omskrevet fra å kalle API-endepunkter (`/auth/login`, `/auth/logout`, `/me`) til å behandle all autentiseringslogikk lokalt i nettleseren. Seed-brukere hardkodes direkte i filen, og nye brukere lagres i `localStorage` under nøkkelen `flowcrt.demo.users`.

**Endringer i `Frontend/src/auth/AuthContext.jsx`:**
- `login()`-funksjonen sjekker nå mot en kombinasjon av hardkodede seed-brukere og brukere lagret i localStorage, i stedet for å sende `POST /auth/login`
- `logout()`-funksjonen tømmer kun den lokale `user`-staten – ingen API-kall
- Sessjonspersistens oppnås ved å lagre innlogget bruker i `localStorage` og lese den tilbake ved sideinnlasting (`useEffect` på montering)
- Seed-brukere definert direkte i filen: én vanlig bruker (`USER`) og én administrator (`FLOW_ADMIN`) for rask testing

**Ny funksjon `register()` i `AuthContext.jsx`:**
- Tar inn `name`, `email` og `password`
- Sjekker om e-postadressen allerede finnes blant seed-brukere og localStorage-brukere
- Lagrer ny bruker som JSON i `localStorage` (`flowcrt.demo.users`-arrayen)
- Setter ny bruker som innlogget umiddelbart etter registrering

**Statistikk:** 1 fil endret, ~60 linjer lagt til, ~40 linjer slettet

---

## Arkitekturbeslutninger

### Hvorfor mock-modus fremfor å bruke et staging-API?

Prosjektet bruker FastAPI-backend som krever at Docker kjører lokalt. For en demo til sensor eller veileder er det uheldig å være avhengig av dette. Mock-modusen gjør at demoen kan kjøres direkte fra `npm run dev` uten ekstra oppsett, noe som drastisk senker terskelen for å vise frem arbeidet.

### Hvorfor localStorage og ikke sessionStorage?

`sessionStorage` tømmes når nettleserfanen lukkes, noe som gjør det vanskelig å demonstrere at data faktisk persisterer på tvers av innlogginger. `localStorage` overlever fane-lukking og gir en mer realistisk illusion av et fungerende system.

### Hvorfor seed-brukere i stedet for kun localStorage?

Seed-brukerne sikrer at en fersk nettleser (uten noe lagret i localStorage) alltid har gyldige testkontoer tilgjengelig. Det unngår situasjoner der en demo feiler fordi localStorage er tomt.

---

## Neste steg
- Oppdatere `useFlowPage.js` til å bruke localStorage for lagring og henting av flowdiagrammer, slik at flow-persistens også fungerer uten backend
