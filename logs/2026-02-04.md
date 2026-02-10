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
