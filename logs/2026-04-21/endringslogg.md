# Endringslogg - Mandag 21. april 2026

## Oversikt
Kjøpssiden (`PurchaseStandards.jsx`) ble bygget om fra et enkelt produktgrid til en 3-stegs wizard der brukeren først velger eller oppretter bedrift, deretter velger produkt, og til slutt ser en bekreftelse. `buyStandard` i `AuthContext.jsx` ble oppdatert til å støtte kjøp for eksisterende bedrifter via `domainId`. Nye CSS-klasser for wizard-komponentene ble lagt til i `styles.css`. Kun frontend-endringer — backend-avhengigheter er dokumentert.

---

## Commits (kronologisk)

### 1. Steg-wizard for kjøp av standarder (ucommitted)

**Tidspunkt:** Ettermiddag

**Endret i `Frontend/src/auth/AuthContext.jsx`:**
- `buyStandard`-signaturen endret fra `(standardCode, companyName)` til `({ productCode, companyName, domainId })`
- Ny logikk: dersom `domainId` er satt (eksisterende bedrift), hoppes `/auth/buy-flow` over og `domain_id` sendes med til `/auth/buy-product`
- Dersom `domainId` ikke er satt (ny bedrift), kalles `/auth/buy-flow` først med `company_name`, deretter `/auth/buy-product`
- Uansett path hentes oppdatert brukerdata fra `/me` etterpå

**Endret i `Frontend/src/pages/PurchaseStandards.jsx` (full omskriving):**
- Erstattet enkelt produktgrid med 3-stegs wizard:
  - **Steg 1 (company):** VISITOR ser tekstinput for bedriftsnavn. FLOW_ADMIN/SYSTEM_ADMIN ser dropdown med eksisterende bedrifter fra `user.access` pluss "Ny bedrift"-alternativ
  - **Steg 2 (product):** Viser produktkort (ISO 9001, 14001, 27001, Tom Canvas). Allerede kjøpte produkter er deaktivert med visuell indikasjon
  - **Steg 3 (done):** Bekreftelsesside med produktnavn, bedriftsnavn og aktiv rolle. Knapper for "Kjøp mer" (tilbake til steg 1) og "Gå til diagram"
- Steg-indikator øverst viser fremdrift med nummererte sirkler og hake for fullførte steg
- `useNavigate` brukes for "Gå til diagram"-knappen som navigerer til `/iso9001`

**Endret i `Frontend/src/styles.css`:**
- Nye CSS-klasser for wizard-komponentene:
  - `.purchase-step-indicator` — flexbox-rad med steg-sirkler
  - `.purchase-step-dot`, `.purchase-step-number`, `.purchase-step-label` — stegindikatorer med active/completed-tilstander
  - `.purchase-wizard-step` — container for hvert steg
  - `.purchase-company-input`, `.purchase-company-select` — inputfelt og dropdown
  - `.purchase-btn-primary`, `.purchase-btn-secondary` — knapper med hover/disabled-tilstander
  - `.purchase-confirmation` — sentrert bekreftelsesside
  - `.purchase-card.purchased` — redusert opacity for kjøpte produkter

---

## Oppsummering
- **3 filer endret** (AuthContext.jsx, PurchaseStandards.jsx, styles.css), ca. 200 linjer lagt til, 50 linjer slettet
- Kjøpsflyten er nå en strukturert wizard i stedet for et flatt grid
- Bedriftsvalg/-opprettelse er integrert som første steg i kjøpsprosessen
- `buyStandard` støtter nå kjøp for eksisterende bedrifter via `domainId`

---

## Begrunnelse

### Hvorfor en steg-wizard i stedet for å beholde det flate gridet?

Det opprinnelige gridet hadde ingen bedriftskontekst — brukeren klikket "Kjøp" på et produkt uten å spesifisere hvilken bedrift det skulle knyttes til. For FLOW_ADMIN-brukere med flere bedrifter er dette tvetydig. En wizard tvinger brukeren til å ta et bevisst valg om bedrift før produktvalg, noe som eliminerer tvetydigheten og gjør det mulig å vise riktig disabled-state per bedrift. Steg-basert UX er også mer i tråd med kjøpsflyten i produksjonsmiljøer (handlekurv-mønsteret).

### Hvorfor objekt-parameter i stedet for posisjonsargumenter?

Den gamle signaturen `buyStandard(standardCode, companyName)` var allerede uoversiktlig med to parametre. Med tillegg av `domainId` ville tre posisjonsargumenter — der noen er valgfrie — blitt vanskelig å lese og feilutsatt. Et objekt `{ productCode, companyName, domainId }` gjør det eksplisitt hvilke verdier som sendes, og er lettere å utvide dersom flere parametre trengs i fremtiden.

---

## Ikke-koderelatert arbeid

I tillegg til kodeendringene ble det brukt tid på:

1. **Planlegging av wizard-implementasjonen** — gjennomgikk brukerflyten for VISITOR vs. FLOW_ADMIN og definerte state-maskinen med 3 steg
2. **Identifisering av backend-avhengigheter** — dokumenterte at `/me` mangler `company_name` i `DomainAccess`-schema, at `/auth/buy-product` trenger valgfri `domain_id`, og at per-domain purchased_products trengs for korrekt disabled-state
