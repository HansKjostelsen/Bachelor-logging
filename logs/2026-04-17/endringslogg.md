# Endringslogg - Fredag 17. april 2026

## Oversikt
Siste bit av demo-funksjonaliteten ble fullført denne dagen. `Register.jsx` ble oppdatert til å bruke `register()`-funksjonen fra `AuthContext`, og `buyStandard()` ble implementert slik at en vanlig bruker kan oppgradere seg selv til `FLOW_ADMIN` ved å "kjøpe" en ISO-standard. Den komplette demo-flyten – registrer, kjøp standard, rediger flow, logg ut og logg inn igjen – ble verifisert å fungere ende til ende med data som persisterer i localStorage.

---

## Commits (kronologisk)

### 1. Register.jsx bruker AuthContext.register() og logger inn automatisk

**Beskrivelse:**
`Register.jsx` sendte tidligere `POST /auth/register` direkte. Filen ble oppdatert til å kalle `useAuth().register()` i stedet, som nå inneholder all registreringslogikk. Etter vellykket registrering navigerer siden automatisk til `/` (forsiden) uten at brukeren trenger å logge inn manuelt.

**Endringer i `Frontend/src/pages/Register.jsx`:**
- Importerer `useAuth` og destrukturerer `register`
- `handleSubmit` kaller `register(name, email, password)` i stedet for `fetch`
- Vellykket registrering navigerer til `/` via `useNavigate` (brukeren er allerede innlogget)
- Feilmelding fra `register()` (f.eks. "E-postadressen er allerede registrert") fanges og vises til brukeren

**Statistikk:** 1 fil endret, ~10 linjer lagt til, ~25 linjer slettet

---

### 2. buyStandard() i AuthContext – rolleoppgradering til FLOW_ADMIN

**Beskrivelse:**
`buyStandard()` ble lagt til i `AuthContext.jsx`. Funksjonen oppgraderer den innloggede brukerens rolle fra `USER` til `FLOW_ADMIN`, oppdaterer sessionen i localStorage og oppdaterer React-staten umiddelbart slik at UI-et reflekterer endringen uten omlasting.

**Endringer i `Frontend/src/auth/AuthContext.jsx`:**
- Ny funksjon `buyStandard()` som leser gjeldende bruker fra `user`-state
- Oppdaterer `role`-feltet til `FLOW_ADMIN` i sessjonsobjektet
- Lagrer oppdatert sesjon tilbake til `localStorage` (SESSION_KEY)
- Oppdaterer i tillegg brukerens post i localStorage-brukerdatabasen (USERS_KEY) slik at rollen persisterer ved neste innlogging
- Setter `user`-state med oppdatert objekt slik at `useAuth().user.role` oppdateres umiddelbart i hele appen
- Eksporteres via `AuthContext`-verdien

**Statistikk:** 1 fil endret, ~20 linjer lagt til

---

## Utfordringer og løsninger

### Rolleendring persisterte ikke ved ny innlogging

Første implementering av `buyStandard()` oppdaterte kun SESSION_KEY i localStorage. Dette fungerte i den pågående sesjonen, men ved neste innlogging ble rollen hentet fra USERS_KEY – og der stod det fortsatt `USER`. Løsningen var å oppdatere brukerposten i USERS_KEY i tillegg til SESSION_KEY, slik at `login()` plukker opp den oppgraderte rollen neste gang.

---

## Verifisert demo-flyt

Følgende ende-til-ende-flyt ble testet og bekreftet å fungere:

1. Åpne app i nettleser (ingen backend kjørende)
2. Naviger til registreringssiden og opprett ny konto
3. Siden navigerer automatisk til forsiden som innlogget bruker (rolle: USER)
4. Trykk "Kjøp ISO-standard" – rollen oppdateres til FLOW_ADMIN umiddelbart
5. Naviger til en ISO 9001-underside – verktøylinje og Lagre-knapp er nå synlig
6. Gjør endringer i flowdiagrammet – Lagre-knappen aktiveres (grønn)
7. Trykk Lagre – knappen deaktiveres igjen (grå)
8. Logg ut og logg inn igjen med samme konto
9. Naviger tilbake til samme underside – diagrammet vises slik det ble lagret

---

## Arkitekturbeslutninger

### Hvorfor oppdatere USERS_KEY i tillegg til SESSION_KEY i buyStandard()?

SESSION_KEY representerer den aktive sesjonen og leses kun ved sideinnlasting. USERS_KEY er "databasen" som `login()` slår opp mot. Uten å oppdatere USERS_KEY ville en bruker som logger ut og inn igjen miste den oppgraderte rollen. Dette er et bevisst valg om å holde localStorage konsistent selv om det er et demo-system.

### Hvorfor automatisk innlogging etter registrering?

Det er unaturlig å registrere en konto og så måtte logge inn manuelt. Alle moderne registreringsflyter logger brukeren inn automatisk. Siden `register()` i AuthContext allerede setter `user`-state etter opprettelse, er dette enkelt å oppnå – `Register.jsx` trenger bare å navigere til `/`.

---

## Neste steg
- Demo-funksjonaliteten er fullstendig. Neste fase er å skrive ferdig bachelor-rapporten med dokumentasjon av arkitektur, tekniske valg og vurdering av arbeidet.
