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

---

## Begrunnelse

Denne seksjonen forklarer hvorfor de tekniske valgene som ble tatt i dag var fornuftige sett i sammenheng med prosjektets behov og fremtidige retning.

### Hvorfor JSONB for lagring av nodes og edges?

React Flow representerer et diagram som to lister: én for noder og én for kanter (edges). Begge listene inneholder objekter med posisjoner, stildata, relasjoner og egendefinerte felt. Det finnes i praksis to måter å lagre dette på:

1. **Normaliserte tabeller** - én rad per node, én rad per kant, med egne kolonner for hvert felt
2. **JSONB-kolonner** - hele listen lagres som ett JSON-objekt per kolonne

Normaliserte tabeller hadde vært mer "korrekt" i relasjonell databaseteori, men i dette tilfellet er det JSONB som gir mest mening. Grunnen er at React Flow allerede opererer med JSON internt - tilstanden leses og skrives som JavaScript-objekter. Med JSONB kan vi lagre og hente tilstanden direkte uten å måtte serialisere og deserialisere hvert enkelt felt via SQL. Det gir enklere kode, færre JOIN-operasjoner, og bedre ytelse for det vi faktisk gjør: å hente hele flowen for én bruker og én side om gangen. PostgreSQL sitt JSONB-format er dessuten indekserbart og validerer JSON-syntaksen ved innsetting, slik at man beholder kontroll uten å ofre fleksibilitet.

### Hvorfor ett flow per bruker per side (UNIQUE constraint)?

Et alternativ hadde vært å tillate at en bruker kan ha flere lagrede flows per side, for eksempel med et versjonssystem. Vi valgte å ikke gjøre det ennå, fordi kravene ikke tilsier det: systemet skal lagre brukerens nåværende tilstand, ikke historikk. `UNIQUE (user_id, page_id)` gjør at en `INSERT ... ON CONFLICT DO UPDATE`-operasjon alltid oppdaterer eksisterende rad istedenfor å lage duplikater, noe som gir en enkel og robust upsert-logikk uten ekstra bookkeping.

### Hvorfor en egen Python-modellklasse (UserFlow)?

Det er mulig å jobbe direkte med dict-objekter fra databasen gjennom hele API-laget, men det er lite robust. En dedikert `UserFlow`-klasse gir tre konkrete fordeler:

- **Klarere grensesnitt**: Koden som bruker modellen vet nøyaktig hvilke felt som finnes, fremfor å navigere i frie dict-strukturer.
- **Sentralisert håndtering av kanttilfeller**: `from_db_row()` håndterer at psycopg2 noen ganger returnerer JSONB som en Python-liste og andre ganger som en JSON-streng, avhengig av driverversjon og konfigurasjon. Denne logikken trengs bare på ett sted.
- **Enkel å utvide**: Validering, transformasjon og tilleggsfelt kan legges til i klassen uten å endre resten av kodebasen.

Bruken av `@dataclass` fra standardbiblioteket gir automatisk `__init__` og `__repr__` uten at vi trenger tredjeparts-biblioteker som Pydantic for dette enkle tilfellet.

### Hvorfor egne hjelpefunksjoner i database.py?

De eksisterende databasekallene i prosjektet hadde innebygget boilerplate for å åpne tilkobling, utføre spørring og returnere den til poolen. Denne koden ble gjentatt for hvert endepunkt, noe som er en typisk kilde til feil - spesielt rundt feilhåndtering, der en unnlatt `putconn()` kan lekke tilkoblinger. Ved å samle denne logikken i `execute_query()` og `execute_write()` sikrer vi at tilkoblinger alltid returneres til poolen via `finally`-blokken, og at commit/rollback håndteres konsekvent for skriveoperasjoner. Det reduserer mengden repetitiv kode i endepunktene og gjør det lettere å teste dem isolert.
