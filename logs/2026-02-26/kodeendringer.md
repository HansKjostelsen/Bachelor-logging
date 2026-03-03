# Kodeendringer - 2026-02-26

Dette dokumentet viser de faktiske kodeendringene som ble gjort 26. februar 2026 i FlowCRT-prosjektet.

**Diff-område:** `3d94149..b71e3a9` (fra handle-fix til flow-modell-commit)

**Merk:** To commits denne dagen. Fokus på databasestruktur og API-modell for brukerlagring av flowdiagrammer.

---

## Hovedendringer

### 1. SQL-migrering - Ny tabell for flow-lagring

#### Database/migration/V4__create_user_flows_table.sql (NY FIL)

```sql
-- Tabell for å lagre React Flow-diagrammer per bruker og ISO-side
CREATE TABLE IF NOT EXISTS user_flows (
    id          SERIAL PRIMARY KEY,
    user_id     INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    page_id     VARCHAR(100) NOT NULL,
    nodes       JSONB NOT NULL DEFAULT '[]',
    edges       JSONB NOT NULL DEFAULT '[]',
    created_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at  TIMESTAMP WITH TIME ZONE DEFAULT NOW(),

    -- Én bruker kan kun ha ett lagret flow per side
    CONSTRAINT uq_user_page UNIQUE (user_id, page_id)
);

-- Indeks for raskere oppslag på user_id
CREATE INDEX IF NOT EXISTS idx_user_flows_user_id ON user_flows(user_id);

-- Indeks for oppslag på page_id (f.eks. 'dokumentstyring', 'kunder')
CREATE INDEX IF NOT EXISTS idx_user_flows_page_id ON user_flows(page_id);
```

**Forklaring:**
- `user_id` er fremmednøkkel til `users`-tabellen med cascade-sletting (slettes automatisk om brukeren slettes)
- `page_id` er en streng som tilsvarer slug-verdien fra React-rutingen (f.eks. `'dokumentstyring'`, `'kunder'`)
- `nodes` og `edges` er JSONB-kolonner som lagrer React Flow-tilstand direkte som JSON-arrays
- `UNIQUE (user_id, page_id)` sikrer at en bruker alltid kun har ett flow per side - oppdatering gjøres med `ON CONFLICT DO UPDATE`
- Indekser på begge søkekolonner for god ytelse

---

### 2. SQL-trigger - Automatisk oppdatering av updated_at

#### Database/migration/V5__add_flow_updated_at_trigger.sql (NY FIL)

```sql
-- Funksjon som setter updated_at til nåværende tidspunkt
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger som kaller funksjonen ved UPDATE på user_flows
CREATE TRIGGER set_updated_at
    BEFORE UPDATE ON user_flows
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();
```

**Forklaring:**
- Funksjonen `update_updated_at_column()` er generisk og kan gjenbrukes på andre tabeller
- Triggeren `set_updated_at` sørger for at `updated_at` alltid reflekterer siste endring uten at klientkoden trenger å sende inn tidsstempel

---

### 3. API-modell for flow-data

#### API/src/models/flow_model.py (NY FIL)

```python
import json
from dataclasses import dataclass, field
from typing import List, Dict, Any


@dataclass
class UserFlow:
    """Representerer et lagret React Flow-diagram for én bruker og én ISO-side."""

    user_id: int
    page_id: str
    nodes: List[Dict[str, Any]] = field(default_factory=list)
    edges: List[Dict[str, Any]] = field(default_factory=list)
    id: int = None
    created_at: str = None
    updated_at: str = None

    @classmethod
    def from_db_row(cls, row: dict) -> "UserFlow":
        """Opprett UserFlow-objekt fra en databaserad."""
        return cls(
            id=row["id"],
            user_id=row["user_id"],
            page_id=row["page_id"],
            nodes=row["nodes"] if isinstance(row["nodes"], list) else json.loads(row["nodes"]),
            edges=row["edges"] if isinstance(row["edges"], list) else json.loads(row["edges"]),
            created_at=str(row.get("created_at", "")),
            updated_at=str(row.get("updated_at", "")),
        )

    def to_dict(self) -> dict:
        """Serialiser til dict for JSON-respons."""
        return {
            "id": self.id,
            "user_id": self.user_id,
            "page_id": self.page_id,
            "nodes": self.nodes,
            "edges": self.edges,
            "created_at": self.created_at,
            "updated_at": self.updated_at,
        }

    def validate(self) -> bool:
        """Enkel validering av at nodes og edges er lister."""
        if not isinstance(self.nodes, list):
            return False
        if not isinstance(self.edges, list):
            return False
        return True
```

**Forklaring:**
- `@dataclass` gir automatisk `__init__` og `__repr__`
- `from_db_row()` håndterer at PostgreSQL-driveren noen ganger returnerer JSONB som streng, andre ganger som Python-objekt
- `to_dict()` brukes for å serialisere til JSON-respons i API-endepunktene
- `validate()` er minimal validering som kan bygges ut etter behov

---

### 4. Forbedringer i database.py

#### API/src/database.py (ENDRET)

```diff
 import psycopg2
 from psycopg2 import pool
+from psycopg2.extras import RealDictCursor
 import os

 connection_pool = None

 def get_db_connection():
     global connection_pool
     if connection_pool is None:
         connection_pool = psycopg2.pool.SimpleConnectionPool(
             1, 20,
             host=os.getenv('DB_HOST', 'localhost'),
             database=os.getenv('DB_NAME', 'isobase'),
             user=os.getenv('DB_USER', 'postgres'),
             password=os.getenv('DB_PASSWORD', 'postgres')
         )
     return connection_pool.getconn()

+
+def execute_query(sql: str, params: tuple = None, fetchone: bool = False):
+    """Hjelpefunksjon for enkle SELECT-spørringer. Returnerer dict-rader."""
+    conn = get_db_connection()
+    try:
+        with conn.cursor(cursor_factory=RealDictCursor) as cursor:
+            cursor.execute(sql, params)
+            if fetchone:
+                return cursor.fetchone()
+            return cursor.fetchall()
+    finally:
+        connection_pool.putconn(conn)
+
+
+def execute_write(sql: str, params: tuple = None) -> int:
+    """Hjelpefunksjon for INSERT/UPDATE/DELETE. Returnerer antall rader påvirket."""
+    conn = get_db_connection()
+    try:
+        with conn.cursor() as cursor:
+            cursor.execute(sql, params)
+            conn.commit()
+            return cursor.rowcount
+    except Exception:
+        conn.rollback()
+        raise
+    finally:
+        connection_pool.putconn(conn)
```

**Forklaring:**
- `RealDictCursor` gjør at rader returneres som Python-dict (`{"id": 1, "user_id": 2, ...}`) istedenfor tupler
- `execute_query()` håndterer connection-pooling og returnering korrekt
- `execute_write()` wrapper commit og rollback rundt skriveoperasjoner
- Begge hjelper med å rydde opp tilkoblinger til poolen etter bruk via `finally`

---

## Visuell effekt

**Ingen synlig endring i frontend ennå** - alle endringene i dag er på database- og API-nivå. Neste steg er å koble disse til faktiske API-endepunkter og deretter et React-hook i frontend for å laste og lagre flowdiagrammer.
