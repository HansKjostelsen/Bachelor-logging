# Kodeendringer - 2026-02-27

Dette dokumentet viser de faktiske kodeendringene som ble gjort 27. februar 2026 i FlowCRT-prosjektet.

**Diff-område:** `b71e3a9..e14a607` (fra flow-modell til React-hook-commit)

**Merk:** To commits denne dagen. Fokus på Flask API-endepunkter og React-hook for flow-persistens.

---

## Hovedendringer

### 1. Flask Blueprint for flow-endepunkter

#### API/src/routes/flow_routes.py (NY FIL)

```python
from flask import Blueprint, request, jsonify
from models.flow_model import UserFlow
from database import execute_query, execute_write

flow_bp = Blueprint("flows", __name__)


@flow_bp.route("/<string:page_id>", methods=["GET"])
def get_flow(page_id: str):
    """Hent lagret flow for innlogget bruker og gitt side."""
    # TODO: Hent user_id fra JWT-token når autentisering er på plass
    user_id = request.args.get("user_id", type=int)
    if not user_id:
        return jsonify({"error": "user_id mangler"}), 400

    row = execute_query(
        "SELECT * FROM user_flows WHERE user_id = %s AND page_id = %s",
        (user_id, page_id),
        fetchone=True,
    )

    if not row:
        # Returner tom tilstand om ingen lagret flow finnes
        return jsonify({"nodes": [], "edges": [], "exists": False}), 200

    flow = UserFlow.from_db_row(row)
    return jsonify({**flow.to_dict(), "exists": True}), 200


@flow_bp.route("/<string:page_id>", methods=["POST"])
def save_flow(page_id: str):
    """Opprett eller oppdater (upsert) et flow for innlogget bruker og gitt side."""
    user_id = request.args.get("user_id", type=int)
    if not user_id:
        return jsonify({"error": "user_id mangler"}), 400

    body = request.get_json()
    if not body:
        return jsonify({"error": "Request-body mangler"}), 400

    nodes = body.get("nodes")
    edges = body.get("edges")

    if not isinstance(nodes, list) or not isinstance(edges, list):
        return jsonify({"error": "nodes og edges må være lister"}), 422

    import json

    execute_write(
        """
        INSERT INTO user_flows (user_id, page_id, nodes, edges)
        VALUES (%s, %s, %s, %s)
        ON CONFLICT (user_id, page_id)
        DO UPDATE SET
            nodes = EXCLUDED.nodes,
            edges = EXCLUDED.edges
        """,
        (user_id, page_id, json.dumps(nodes), json.dumps(edges)),
    )

    return jsonify({"message": "Flow lagret"}), 200


@flow_bp.route("/<string:page_id>", methods=["DELETE"])
def delete_flow(page_id: str):
    """Slett lagret flow for innlogget bruker og gitt side."""
    user_id = request.args.get("user_id", type=int)
    if not user_id:
        return jsonify({"error": "user_id mangler"}), 400

    rows_affected = execute_write(
        "DELETE FROM user_flows WHERE user_id = %s AND page_id = %s",
        (user_id, page_id),
    )

    if rows_affected == 0:
        return jsonify({"error": "Ingen flow funnet"}), 404

    return jsonify({"message": "Flow slettet"}), 200
```

**Forklaring:**
- Blueprint separerer flow-logikken fra resten av API-et og gjør koden lettere å vedlikeholde
- `ON CONFLICT (user_id, page_id) DO UPDATE` implementerer upsert - én operasjon håndterer både opprettelse og oppdatering
- `user_id` hentes foreløpig fra query-param; skal erstattes med JWT-token-parsing når autentisering er implementert
- Returner HTTP 422 ved feil dataformat, 404 om flow ikke finnes

---

### 2. Registrering av Blueprint i Main.py

#### API/src/Main.py (ENDRET)

```diff
 from flask import Flask, jsonify, request
 from flask_cors import CORS
 from database import get_db_connection
 from app_config import Config
+from routes.flow_routes import flow_bp

 app = Flask(__name__)
 app.config.from_object(Config)
 CORS(app)

+# Registrer Blueprint for flow-endepunkter
+app.register_blueprint(flow_bp, url_prefix="/api/flows")

 @app.route('/api/health', methods=['GET'])
 def health_check():
     return jsonify({"status": "healthy", "message": "API is running"}), 200
```

**Forklaring:**
- `url_prefix="/api/flows"` gjør at alle ruter i Blueprint-et prefixeres med `/api/flows`
- Endelig URL-struktur: `GET /api/flows/dokumentstyring`, `POST /api/flows/kunder`, osv.

---

### 3. React-hook for flow-persistens

#### Frontend/src/hooks/useFlowPersistence.js (NY FIL)

```javascript
import { useState, useCallback } from "react";

const API_BASE = import.meta.env.VITE_API_URL || "http://localhost:1337";

/**
 * Custom hook for å laste og lagre React Flow-diagrammer via API.
 * @param {string} pageId - Slug-identifikator for siden (f.eks. 'dokumentstyring')
 * @param {number} userId - ID til innlogget bruker
 */
export function useFlowPersistence(pageId, userId) {
  const [isSaving, setIsSaving] = useState(false);
  const [lastSaved, setLastSaved] = useState(null);
  const [error, setError] = useState(null);

  const loadFlow = useCallback(async () => {
    try {
      const res = await fetch(
        `${API_BASE}/api/flows/${pageId}?user_id=${userId}`
      );
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      const data = await res.json();
      return { nodes: data.nodes, edges: data.edges, exists: data.exists };
    } catch (err) {
      setError(err.message);
      return { nodes: [], edges: [], exists: false };
    }
  }, [pageId, userId]);

  const saveFlow = useCallback(
    async (nodes, edges) => {
      setIsSaving(true);
      setError(null);
      try {
        const res = await fetch(
          `${API_BASE}/api/flows/${pageId}?user_id=${userId}`,
          {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ nodes, edges }),
          }
        );
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        setLastSaved(new Date());
      } catch (err) {
        setError(err.message);
      } finally {
        setIsSaving(false);
      }
    },
    [pageId, userId]
  );

  return { loadFlow, saveFlow, isSaving, lastSaved, error };
}
```

**Forklaring:**
- `import.meta.env.VITE_API_URL` lar API-URLen konfigureres via miljøvariabel, med fallback til localhost
- `useCallback` med avhengigheter sikrer at `loadFlow` og `saveFlow` ikke gjenskapes unødvendig
- `isSaving` kan brukes til å vise spinner eller deaktivere "Lagre"-knapp mens lagring pågår
- `lastSaved` kan vises i UI-et som "Sist lagret: [tidspunkt]"

---

### 4. Integrering i Dokumentstyring (proof-of-concept)

#### Frontend/src/pages/ISO-sider/9001-Undersider/Dokumentstyring.jsx (ENDRET)

```diff
+import { useEffect } from 'react';
 import { useState, useCallback } from 'react';
 import { ReactFlow, applyNodeChanges, applyEdgeChanges, MarkerType } from '@xyflow/react';
 import '@xyflow/react/dist/style.css';
 import FlowCRTNode from '../../../components/FlowCRTNode';
 import { useNavigate } from 'react-router-dom';
+import { useFlowPersistence } from '../../../hooks/useFlowPersistence';

 export default function Dokumentstyring() {
   const [nodes, setNodes] = useState(initialNodes);
   const [edges, setEdges] = useState(initialEdges);
   const navigate = useNavigate();

+  // TODO: Bytt ut hardkodet userId=1 med faktisk innlogget bruker
+  const { loadFlow, saveFlow, isSaving, lastSaved } = useFlowPersistence('dokumentstyring', 1);
+
+  useEffect(() => {
+    loadFlow().then(({ nodes: savedNodes, edges: savedEdges, exists }) => {
+      if (exists) {
+        setNodes(savedNodes);
+        setEdges(savedEdges);
+      }
+    });
+  }, []);

   // ...existing handlers...

   return (
     <div className="ISO-side">
       <button className="back-button" onClick={() => navigate('/iso9001')}>
         ← Tilbake til ISO 9001
       </button>
       <h1 className="ISO-tittel">Dokumentstyring</h1>

+      <div className="flow-actions">
+        <button
+          className="save-button"
+          onClick={() => saveFlow(nodes, edges)}
+          disabled={isSaving}
+        >
+          {isSaving ? 'Lagrer...' : 'Lagre diagram'}
+        </button>
+        {lastSaved && (
+          <span className="last-saved">
+            Sist lagret: {lastSaved.toLocaleTimeString('nb-NO')}
+          </span>
+        )}
+      </div>

       <div className="flow-wrapper">
         <ReactFlow
           nodes={nodes}
           edges={edges}
```

**Forklaring:**
- `useEffect` med tom avhengighetsliste kjøres én gang ved montering og laster inn sist lagret tilstand
- Dersom ingen lagret flow finnes (`exists: false`), beholdes `initialNodes` og `initialEdges`
- "Lagre diagram"-knappen er deaktivert mens lagring pågår for å hindre dobbelt-klikk
- `toLocaleTimeString('nb-NO')` formaterer tidsstempelet på norsk

---

## Oppsummering av endringer

- **5 filer endret**, 187 linjer lagt til, 8 linjer slettet
- Flask Blueprint med tre endepunkter: GET, POST (upsert) og DELETE for `/api/flows/<page_id>`
- Custom React-hook `useFlowPersistence` for gjenbrukbar flow-persistens
- Proof-of-concept i Dokumentstyring viser at hele flyten (last inn → rediger → lagre) fungerer
- Neste steg er JWT-autentisering og å rulle ut persistens til de øvrige undersidene
