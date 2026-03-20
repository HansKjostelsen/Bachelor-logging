# Ukeslogg - Uke 9 (10. mars - 12. mars 2026)

## Periode: Tirsdag 10. - Torsdag 12. mars 2026

---

## Sammendrag

Uke 9 var primært viet til å legge grunnlaget for brukerlagring av flowdiagrammer i FlowCRT. Arbeidet startet med feilsøking og fikse av et irriterende CSS-problem med handle-plassering tirsdag, og fortsatte med systematisk design og implementasjon av databasestruktur, API-endepunkter og frontend-integrasjon for å persistere React Flow-diagrammer per bruker. Ved ukens slutt hadde systemet en fungerende ende-til-ende-løsning for lagring og lasting av flow-tilstand, demonstrert som proof-of-concept på Dokumentstyring-siden.

---

## Dag for dag

### Tirsdag 10. mars
Debugget og fikset et CSS-problem i `styles.css` der regelen `top: 51px !important` tvang alle React Flow-handles til feil posisjon. Etter at feilen var løst ble det brukt tid på å kartlegge hva som trengs for å støtte brukerlagring av flowdiagrammer - hvilke tabeller, kolonner og API-endepunkter som mangler.

**Commits:** `21803dc` (merge PR #11), `3d94149` (handle-fix)

### Onsdag 11. mars
Designet og implementerte databasestrukturen for flow-lagring. En ny migrasjonsscript `V4__create_user_flows_table.sql` opprettet tabellen `user_flows` med JSONB-kolonner for `nodes` og `edges`, fremmednøkkelkobling mot `users`, og et unikt constraint per bruker og side. En trigger (`V5`) sørger for automatisk oppdatering av `updated_at`. På API-siden ble modellklassen `UserFlow` og hjelpefunksjoner for databasekall lagt til.

**Commits:** `a4f92c1` (DB-migrering), `b71e3a9` (API-modell)

### Torsdag 12. mars
Implementerte Flask API-endepunkter for GET, POST og DELETE av flow-data, organisert i en Blueprint under `/api/flows`. Upsert-logikk via `ON CONFLICT DO UPDATE` gjør at POST håndterer både opprettelse og oppdatering. Et custom React-hook `useFlowPersistence` ble opprettet i frontend for gjenbrukbar lasting og lagring, og integrert i Dokumentstyring-siden som proof-of-concept.

**Commits:** `c93d812` (API-endepunkter), `e14a607` (React-hook + frontend-integrasjon)

---

## Tekniske milepæler denne uken

| Milepæl | Status |
|---------|--------|
| CSS-handle-bug identifisert og fikset | Fullfort |
| Databasetabell `user_flows` med JSONB | Fullfort |
| SQL-trigger for automatisk `updated_at` | Fullfort |
| Python-modellklasse `UserFlow` | Fullfort |
| Flask Blueprint med GET/POST/DELETE | Fullfort |
| React-hook `useFlowPersistence` | Fullfort |
| Proof-of-concept i Dokumentstyring | Fullfort |

---

## Viktige beslutninger

- **JSONB fremfor separate tabellrader:** Valgte å lagre `nodes` og `edges` som JSONB-kolonner fremfor å normalisere til separate rader, da React Flow allerede opererer med JSON-lister og kompleksiteten ikke er verdt gevinsten på dette stadiet.
- **Manuell lagring fremfor autosave:** Landet på en eksplisitt "Lagre diagram"-knapp fremfor debounced autosave for å holde løsningen enkel og unngå unødvendige API-kall under redigering.
- **Upsert-pattern:** POST-endepunktet bruker `ON CONFLICT DO UPDATE` for å håndtere både opprettelse og oppdatering i én operasjon, noe som forenkler frontend-koden.

---

## Hva gjenstår

- JWT-autentisering i API-et (foreløpig er `user_id` en query-param)
- Rulle ut `useFlowPersistence`-hooken til alle de øvrige ISO 9001-undersidene (13 sider igjen)
- CSS-styling av lagre-knapp og "Sist lagret"-indikatoren
- Feilhåndtering i frontend dersom API-et er nede
