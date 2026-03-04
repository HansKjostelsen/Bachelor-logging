# Endringslogg - Onsdag 4. februar 2026

## Oversikt
Frontend-applikasjonen ble opprettet fra scratch, en gammel mappestruktur ble slettet og erstattet, og farger ble justert.

---

## Commits (kronologisk)

### 1. Opprettet applikasjonen (`3748e98`)

**Lagt til:**
- Opprettet Vite + React-prosjekt i `Frontend/Websiden/`
- Standardfiler: `.gitignore`, `package.json`, `eslint.config.js`, `vite.config.js`, `index.html`
- `App.jsx`, `App.css`, `index.css`, `main.jsx`
- 13 nye filer totalt

### 2. La til nodes (`f90d06f`)

**Lagt til:**
- node_modules og diverse avhengigheter (stor commit med pakkefiler)

### 3. Sletta alt (`82de30f`)

**Fjernet:**
- Ryddet opp i filer som ikke skulle vaert med

### 4. Fiksa (`6eca234`)

- Fikset opp etter opprydding

### 5. La inn i frontend (`aba3622`)

- Flyttet filer inn i riktig Frontend-mappestruktur

### 6. Modifisert litt (`07aa64b`)

- Mindre justeringer

### 7. Delete Frontend/Websiden directory (`e7479e8`)

**Fjernet:**
- Hele `Frontend/Websiden/`-mappen (14 filer) - den gamle mappestrukturen ble slettet
- Inkluderte `.gitignore`, `README.md`, `eslint.config.js`, `index.html`, `package-lock.json`, `package.json`, alle src-filer, og `vite.config.js`

### 8. Add initial README for Frontend directory (`2b34dec`)

**Lagt til:**
- `Frontend/Readme.md` - Ny README-fil

### 9. Update README.md (`61cd654`)

**Endret:**
- `Frontend/README.md` - Fjernet 18 linjer med auto-generert Vite-innhold, erstattet med 1 linje

### 10. Bytta farger (`940ebe8`)

**Endret i `Frontend/src/styles.css`:**
- Header bakgrunnsfarge: `#123223` -> `#013220`
- Footer bakgrunnsfarge: `#123223` -> `#013220`

---

## Oppsummering
- Frontend-applikasjonen ble opprettet, ryddet opp, og flyttet til riktig mappestruktur
- Gammel `Websiden/`-mappe ble slettet og erstattet
- Header/footer-farger ble endret fra `#123223` til `#013220` (morkere gronn)
- README ble opprettet og ryddet opp

---

## Begrunnelse

Valget om å bruke **Vite + React** som fundament for frontend-applikasjonen var naturlig gitt prosjektets behov for en moderne, komponentbasert arkitektur. Vite gir svært rask utviklingsserver og hot module replacement, noe som gjør iterasjonssyklusene kortere i en tidlig utviklingsfase der mye skal utforskes og testes.

Bruken av **React Router (`react-router-dom`)** fra starten av sikrer at applikasjonen kan vokse med klart definerte ruter per ISO-standard. Å etablere denne strukturen tidlig — med egne sider for ISO 9001, ISO 14001 og ISO 27001 — gjør det enklere å bygge videre uten å måtte refaktorere routingen i ettertid.

Den **midlertidige oppryddingen** (sletting av `Frontend/Websiden/` og omstrukturering) kan virke som unødvendig frem-og-tilbake, men dette er en vanlig del av tidlig prosjektoppsett der man finner riktig mappekonvensjon. Det er bedre å rydde opp i mappestrukturen tidlig enn å leve med feil kataloghierarki gjennom hele prosjektet.

**Fargevalget** — mørk grønn (`#013220`) i header og footer fremfor den opprinnelige `#123223` — ble justert for å gi et mer profesjonelt og distinkt visuelt uttrykk. Denne typen estetisk finjustering tidlig i prosessen hjelper teamet å enes om en visuell profil før innholdet bygges ut.

README-filen ble bevisst forenklet til én linje istedenfor å beholde den auto-genererte Vite-teksten, fordi autogenerert innhold er misvisende — det beskriver ikke prosjektet, men verktøyet. En kortfattet og ærlig README er mer nyttig enn en lang boilerplate.
