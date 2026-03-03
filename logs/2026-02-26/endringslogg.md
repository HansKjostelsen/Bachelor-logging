# Endringslogg - Onsdag 26. februar 2026

## Oversikt
Dagen ble brukt til å designe og implementere databasestrukturen som trengs for å lagre React Flow-diagrammer per bruker i FlowCRT. En ny migrasjonsscript og tilhørende databasemodell ble opprettet, og eksisterende API-kode ble forberedt for flow-lagring.

---

## Commits (kronologisk)

### 1. Databasemigrering for flow-lagring (`a4f92c1`)

**Tidspunkt:** 10:14

**Ny fil: `Database/migration/V4__create_user_flows_table.sql`:**
- Opprettet ny tabell `user_flows` for å lagre React Flow-diagrammer per bruker og prosessside
- Tabellen lagrer `nodes` og `edges` som JSON-kolonner (JSONB i PostgreSQL)
- Lagt til fremmednøkkelrelasjon mot eksisterende `users`-tabell
- Lagt til `page_id`-kolonne for å identifisere hvilken ISO-side flowet tilhører
- Unikt constraint på `(user_id, page_id)` slik at hver bruker kun har ett flow per side
- `created_at` og `updated_at` med automatisk tidsstempel via trigger

**Ny fil: `Database/migration/V5__add_flow_updated_at_trigger.sql`:**
- Opprettet PostgreSQL-funksjon `update_updated_at_column()` som automatisk setter `updated_at` ved oppdatering
- Lagt til trigger `set_updated_at` på `user_flows`-tabellen

### 2. API-modell for flow-data (`b71e3a9`)

**Tidspunkt:** 13:47

**Ny fil: `API/src/models/flow_model.py`:**
- Opprettet Python-klasse `UserFlow` som representerer en rad i `user_flows`-tabellen
- Metoder for serialisering til og deserialisering fra JSON
- Validering av `nodes` og `edges`-strukturen som React Flow forventer

**Endret i `API/src/database.py`:**
- Lagt til hjelpefunksjon `execute_query()` for å forenkle databasekall
- Lagt til `execute_many()` for batch-operasjoner

---

## Oppsummering
- **4 filer endret/opprettet**, 98 linjer lagt til, 3 linjer slettet
- Ny `user_flows`-tabell med JSONB-kolonner for nodes og edges
- Trigger for automatisk oppdatering av `updated_at`
- Grunnleggende modellklasse på API-siden klar til bruk

---

## Ikke-koderelatert arbeid

I tillegg til kodeendringene ble det brukt tid på å:

1. **Tegne ER-diagram** for den nye tabellstrukturen og avklare relasjonen mellom `users` og `user_flows`
2. **Diskutere lagringsformat** - vurderte å lagre nodes og edges som separate tabellrader, men landet på JSONB-kolonner for enkelhet og ytelse siden React Flow allerede jobber med JSON-lister
3. **Planlegge API-endepunktene** som skal leses og skrives mot den nye tabellen - GET, POST og PUT for `/api/flows/:pageId`
