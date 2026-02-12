# Kodeendringer - 2026-02-12

Dette dokumentet viser de faktiske kodeendringene som ble gjort 12. februar 2026 i FlowCRT-prosjektet.

**Diff-område:** `711b83f..3aeef14` (fra første til siste commit denne dagen)

**Merk:** Dette var en massiv dag med 39 filer endret, 3186+ linjer lagt til. Dette dokumentet viser de viktigste endringene.

---

## Hovedendringer

### 1. FlowCRTNode - Redigerbarhet og multi-directional handles

#### Frontend/src/components/FlowCRTNode.jsx

**Lagt til dobbeltklikk-redigering:**

```diff
-export default function FlowCRTNode({ data }) {
+export default function FlowCRTNode({ data, id }) {
   const [isHovered, setIsHovered] = useState(false);
   const navigate = useNavigate();

   const handleClick = () => {
     if (data.slug) {
       navigate(`/iso9001/${data.slug}`);
     }
   };

+  const handleDoubleClick = (e) => {
+    // Only allow editing if this is a user-created node
+    if (data.editable && data.onLabelChange) {
+      e.stopPropagation(); // Prevent navigation
+      const newLabel = window.prompt('Endre nodenavn:', data.label);
+      if (newLabel && newLabel.trim()) {
+        data.onLabelChange(id, newLabel.trim());
+      }
+    }
+  };
+
+  const handleDelete = (e) => {
+    e.stopPropagation(); // Prevent navigation
+    if (data.onDelete && window.confirm(`Er du sikker på at du vil slette "${data.label}"?`)) {
+      data.onDelete(id);
+    }
+  };

   return (
     <div
       className="flowcrt-node"
       onMouseEnter={() => setIsHovered(true)}
       onMouseLeave={() => setIsHovered(false)}
       onClick={handleClick}
+      onDoubleClick={handleDoubleClick}
     >
```

**Lagt til slette-knapp:**

```diff
+      {/* Show delete button on hover for user-created nodes */}
+      {isHovered && data.editable && data.onDelete && (
+        <button
+          className="node-delete-btn"
+          onClick={handleDelete}
+          title="Slett node"
+        >
+          ×
+        </button>
+      )}
```

**Lagt til multi-directional handles (8 handles):**

```diff
-      <Handle type="target" position={Position.Left} id="left" />
-      <Handle type="source" position={Position.Right} id="right" />
+      {data.showHandles !== false && data.allHandles && (
+        <>
+          <Handle type="target" position={Position.Top} id="top-target" />
+          <Handle type="source" position={Position.Top} id="top-source" />
+          <Handle type="target" position={Position.Bottom} id="bottom-target" />
+          <Handle type="source" position={Position.Bottom} id="bottom-source" />
+          <Handle type="target" position={Position.Left} id="left-target" />
+          <Handle type="source" position={Position.Left} id="left-source" />
+          <Handle type="target" position={Position.Right} id="right-target" />
+          <Handle type="source" position={Position.Right} id="right-source" />
+        </>
+      )}
+
+      {data.showHandles !== false && !data.allHandles && (
+        <>
+          <Handle type="target" position={Position.Left} id="left-target" />
+          <Handle type="source" position={Position.Right} id="right-source" />
+        </>
+      )}
```

**Forklaring:**
- Redigering via `window.prompt()` ved dobbeltklikk (kun for `editable` noder)
- Slette-knapp vises ved hover (krever `editable` og `onDelete` callback)
- To handle-moduser:
  - Standard: 2 handles (left-target, right-source)
  - allHandles: 8 handles (top/bottom/left/right, hver med target og source)
- `e.stopPropagation()` forhindrer navigasjon under redigering/sletting

---

### 2. LabelNode - Ny komponent for tekst uten handles

#### Frontend/src/components/LabelNode.jsx (NY FIL)

```jsx
export default function LabelNode({ data }) {
  return (
    <div className="label-node">
      {data.label}
    </div>
  );
}
```

**Forklaring:**
- Minimalistisk komponent kun for visning av tekst
- Ingen handles, ingen interaktivitet
- Bruksområde: Overskrifter og organisatoriske elementer i flows

---

### 3. Nye ISO 9001-undersider (5 stk)

Alle 5 nye undersider følger samme struktur med realistic prosessnoder.

#### Eksempel: Frontend/src/pages/ISO-sider/9001-Undersider/Dokumentstyring.jsx

```jsx
import { useState, useCallback } from 'react';
import { ReactFlow, applyNodeChanges, applyEdgeChanges, MarkerType } from '@xyflow/react';
import '@xyflow/react/dist/style.css';
import FlowCRTNode from '../../../components/FlowCRTNode';
import { useNavigate } from 'react-router-dom';

const nodeTypes = {
  flowcrt: FlowCRTNode,
};

const initialNodes = [
  {
    id: 'dok-identifisere',
    type: 'flowcrt',
    position: { x: 100, y: 50 },
    data: { label: 'Identifisere behov' },
    draggable: false,
  },
  {
    id: 'dok-opprette',
    type: 'flowcrt',
    position: { x: 300, y: 50 },
    data: { label: 'Opprette dokument' },
    draggable: false,
  },
  {
    id: 'dok-gjennomgaa',
    type: 'flowcrt',
    position: { x: 500, y: 50 },
    data: { label: 'Gjennomgå' },
    draggable: false,
  },
  {
    id: 'dok-godkjenne',
    type: 'flowcrt',
    position: { x: 700, y: 50 },
    data: { label: 'Godkjenne' },
    draggable: false,
  },
  {
    id: 'dok-distribuere',
    type: 'flowcrt',
    position: { x: 100, y: 200 },
    data: { label: 'Distribuere' },
    draggable: false,
  },
  {
    id: 'dok-bruke',
    type: 'flowcrt',
    position: { x: 300, y: 200 },
    data: { label: 'Bruke dokument' },
    draggable: false,
  },
  {
    id: 'dok-oppdatere',
    type: 'flowcrt',
    position: { x: 500, y: 200 },
    data: { label: 'Oppdatere' },
    draggable: false,
  },
  {
    id: 'dok-arkivere',
    type: 'flowcrt',
    position: { x: 700, y: 200 },
    data: { label: 'Arkivere' },
    draggable: false,
  },
  {
    id: 'dok-avvik',
    type: 'flowcrt',
    position: { x: 100, y: 350 },
    data: { label: 'Avvikshåndtering' },
    draggable: false,
  },
  {
    id: 'dok-tilbaketrekke',
    type: 'flowcrt',
    position: { x: 300, y: 350 },
    data: { label: 'Tilbaketrekke' },
    draggable: false,
  },
  {
    id: 'dok-makulere',
    type: 'flowcrt',
    position: { x: 500, y: 350 },
    data: { label: 'Makulere' },
    draggable: false,
  },
  {
    id: 'dok-audit',
    type: 'flowcrt',
    position: { x: 700, y: 350 },
    data: { label: 'Audit' },
    draggable: false,
  },
];

const initialEdges = [
  {
    id: 'dok-e1',
    source: 'dok-identifisere',
    target: 'dok-opprette',
    type: 'straight',
    style: { stroke: 'black', strokeWidth: 2 },
    markerEnd: { type: MarkerType.ArrowClosed, color: 'black' },
  },
  // ... flere edges
];

export default function Dokumentstyring() {
  const [nodes, setNodes] = useState(initialNodes);
  const [edges, setEdges] = useState(initialEdges);
  const navigate = useNavigate();

  const onNodesChange = useCallback(
    (changes) => setNodes((nodesSnapshot) => applyNodeChanges(changes, nodesSnapshot)),
    [],
  );

  const onEdgesChange = useCallback(
    (changes) => setEdges((edgesSnapshot) => applyEdgeChanges(changes, edgesSnapshot)),
    [],
  );

  return (
    <div className="ISO-side">
      <button className="back-button" onClick={() => navigate('/iso9001')}>
        ← Tilbake til ISO 9001
      </button>
      <h1 className="ISO-tittel">Dokumentstyring</h1>

      <div className="flow-wrapper">
        <ReactFlow
          nodes={nodes}
          edges={edges}
          onNodesChange={onNodesChange}
          onEdgesChange={onEdgesChange}
          nodeTypes={nodeTypes}
          proOptions={{ hideAttribution: true }}
          fitView
        />
      </div>
    </div>
  );
}
```

**De 5 nye undersidene:**

1. **Dokumentstyring.jsx** (188 linjer, 12 noder):
   - Identifisere behov → Opprette → Gjennomgå → Godkjenne
   - Distribuere → Bruke → Oppdatere → Arkivere
   - Avvikshåndtering → Tilbaketrekke → Makulere → Audit

2. **InternkontrollOgHMS.jsx** (197 linjer, 15 noder):
   - Planlegge → Identifisere farer → Risikovurdering → Fastsette tiltak
   - Implementere tiltak → Overvåke → Rapportere → Forbedre
   - HMS-opplæring, Beredskap, Hendelser, Etterlevelse

3. **Leverandoerer.jsx** (197 linjer, 15 noder):
   - Identifisere behov → Søk → Kvalifisering → Evaluering
   - Kontrakt → Bestilling → Mottak → Kvalitetskontroll
   - Evaluere ytelse → Oppfølging → Avvik → Forbedring

4. **MaalingOgAnalyse.jsx** (188 linjer, 12 noder):
   - Definere mål → Samle data → Analysere → Rapportere
   - Identifisere avvik → Tiltak → Verifisere → Lukke loop
   - Dashboard, Trender, Benchmarking

5. **StrategiOgStyring.jsx** (197 linjer, 17 noder):
   - Visjon/Misjon → Strategisk planlegging → Ressursallokering → KPI
   - Implementering → Overvåking → Ledergjennomgang → Justering
   - Kommunikasjon, Risikostyring, Compliance, Innovasjon, Stakeholder

---

### 4. ISO9001.jsx - Lagt til 5 nye noder

#### Frontend/src/pages/ISO-sider/ISO9001.jsx

```diff
 const initialNodes = [
   // ... eksisterende 9 noder ...
+  {
+    id: 'n10',
+    type: 'flowcrt',
+    position: { x: 700, y: 200 },
+    data: {
+      label: 'Dokumentstyring',
+      description: 'Håndtering av dokumenter og kvalitetssystem',
+      slug: 'dokumentstyring'
+    },
+    draggable: false,
+  },
+  {
+    id: 'n11',
+    type: 'flowcrt',
+    position: { x: 1050, y: 200 },
+    data: {
+      label: 'Internkontroll og HMS',
+      description: 'Risikovurdering, HMS og compliance',
+      slug: 'internkontroll-og-hms'
+    },
+    draggable: false,
+  },
+  {
+    id: 'n12',
+    type: 'flowcrt',
+    position: { x: 1400, y: 200 },
+    data: {
+      label: 'Leverandører',
+      description: 'Leverandørstyring og evaluering',
+      slug: 'leverandoerer'
+    },
+    draggable: false,
+  },
+  {
+    id: 'n13',
+    type: 'flowcrt',
+    position: { x: 1750, y: 200 },
+    data: {
+      label: 'Måling og Analyse',
+      description: 'KPI, datamåling og forbedringsarbeid',
+      slug: 'maaling-og-analyse'
+    },
+    draggable: false,
+  },
+  {
+    id: 'n14',
+    type: 'flowcrt',
+    position: { x: 2100, y: 200 },
+    data: {
+      label: 'Strategi og Styring',
+      description: 'Ledelsessystem og strategisk planlegging',
+      slug: 'strategi-og-styring'
+    },
+    draggable: false,
+  },
 ];
```

**Forklaring:**
- Totalt 14 noder i hovedvisningen (9 eksisterende + 5 nye)
- Ny rad med noder på y=200 (under den første raden)
- Alle har slug for navigasjon og description for hover-visning

---

### 5. App.jsx - Routing for nye undersider

#### Frontend/src/App.jsx

```diff
+import Dokumentstyring from "./pages/ISO-sider/9001-Undersider/Dokumentstyring";
+import InternkontrollOgHMS from "./pages/ISO-sider/9001-Undersider/InternkontrollOgHMS";
+import Leverandoerer from "./pages/ISO-sider/9001-Undersider/Leverandoerer";
+import MaalingOgAnalyse from "./pages/ISO-sider/9001-Undersider/MaalingOgAnalyse";
+import StrategiOgStyring from "./pages/ISO-sider/9001-Undersider/StrategiOgStyring";
+import Register from "./pages/Register";

 function App() {
   return (
     <BrowserRouter>
       <div className="App">
         <Header />
         <Routes>
           {/* ... eksisterende routes ... */}
+          <Route path="/iso9001/dokumentstyring" element={<Dokumentstyring />} />
+          <Route path="/iso9001/internkontroll-og-hms" element={<InternkontrollOgHMS />} />
+          <Route path="/iso9001/leverandoerer" element={<Leverandoerer />} />
+          <Route path="/iso9001/maaling-og-analyse" element={<MaalingOgAnalyse />} />
+          <Route path="/iso9001/strategi-og-styring" element={<StrategiOgStyring />} />
+          <Route path="/register" element={<Register />} />
         </Routes>
         <Footer />
       </div>
     </BrowserRouter>
   );
 }
```

---

### 6. API - Flask backend med database

#### API/src/Main.py (NY FIL)

```python
from flask import Flask, jsonify, request
from flask_cors import CORS
from database import get_db_connection
from app_config import Config

app = Flask(__name__)
app.config.from_object(Config)
CORS(app)

@app.route('/api/health', methods=['GET'])
def health_check():
    return jsonify({"status": "healthy", "message": "API is running"}), 200

@app.route('/api/users', methods=['GET'])
def get_users():
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users")
    users = cursor.fetchall()
    cursor.close()
    conn.close()
    return jsonify(users), 200

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.get_json()
    # ... user creation logic
    return jsonify({"message": "User created"}), 201

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=1337, debug=True)
```

#### API/src/database.py (NY FIL)

```python
import psycopg2
from psycopg2 import pool
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
```

#### API/src/app_config.py (NY FIL)

```python
import os

class Config:
    SECRET_KEY = os.getenv('SECRET_KEY', 'dev-secret-key')
    DEBUG = os.getenv('DEBUG', 'True') == 'True'
    SQLALCHEMY_DATABASE_URI = os.getenv('DATABASE_URL', 'postgresql://postgres:postgres@localhost/isobase')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
```

---

### 7. CSS - Massiv utvidelse (659+ nye linjer)

#### Frontend/src/styles.css

**Node delete-button styling:**

```css
.node-delete-btn {
  position: absolute;
  top: -8px;
  right: -8px;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #ff4444;
  color: white;
  border: 2px solid white;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
  z-index: 10;
}

.node-delete-btn:hover {
  background: #cc0000;
  transform: scale(1.1);
}
```

**Label node styling:**

```css
.label-node {
  background: transparent;
  color: #002d00;
  padding: 8px 16px;
  font-size: 14px;
  font-weight: 600;
  text-align: center;
}
```

**Flow-wrapper høyde-endring (fra siste commit):**

```diff
 .flow-wrapper {
-  height: 500px;
+  height: 750px;
   border: 3px solid #002d00;
   border-radius: 16px;
   background: #c9c9c6;
   overflow: hidden;
 }
```

**Form-styling (for login/register):**

```css
form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  max-width: 400px;
  margin: 2rem auto;
  padding: 2rem;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

input[type="text"],
input[type="email"],
input[type="password"] {
  padding: 12px;
  border: 2px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
  transition: border-color 0.3s;
}

input:focus {
  outline: none;
  border-color: #013220;
}

button[type="submit"] {
  background: #4CAF50;
  color: white;
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.3s, transform 0.2s;
}

button[type="submit"]:hover {
  background: #45a049;
  transform: translateY(-2px);
}
```

---

### 8. Register-side

#### Frontend/src/pages/Register.jsx (NY FIL)

```jsx
import React from "react";

function Register() {
  return (
    <main>
      <h2>Registrer deg</h2>
      <form>
        <input type="text" placeholder="Brukernavn" required />
        <input type="email" placeholder="E-post" required />
        <input type="password" placeholder="Passord" required />
        <input type="password" placeholder="Bekreft passord" required />
        <button type="submit">Registrer</button>
      </form>
      <p style={{ textAlign: 'center', marginTop: '1rem' }}>
        Har du allerede en konto? <a href="/login">Logg inn her</a>
      </p>
    </main>
  );
}

export default Register;
```

---

### 9. Oppdaterte eksisterende undersider

Alle 9 eksisterende undersider ble oppgradert fra placeholders til realistic prosessnoder.

**Eksempel: Anskaffelse.jsx - før og etter**

**Før (3-4 placeholder-noder):**
```jsx
const initialNodes = [
  { id: 'ansk1', type: 'flowcrt', position: { x: 0, y: 0 }, data: { label: 'Node 1' } },
  { id: 'ansk2', type: 'flowcrt', position: { x: 200, y: 0 }, data: { label: 'Node 2' } },
];
```

**Etter (15+ realistic noder):**
```jsx
const initialNodes = [
  { id: 'ansk-behov', type: 'flowcrt', position: { x: 100, y: 50 }, data: { label: 'Identifisere behov' } },
  { id: 'ansk-spesifikasjon', type: 'flowcrt', position: { x: 300, y: 50 }, data: { label: 'Utarbeide spesifikasjon' } },
  { id: 'ansk-leverandor', type: 'flowcrt', position: { x: 500, y: 50 }, data: { label: 'Velge leverandør' } },
  { id: 'ansk-tilbud', type: 'flowcrt', position: { x: 700, y: 50 }, data: { label: 'Innhente tilbud' } },
  { id: 'ansk-evaluering', type: 'flowcrt', position: { x: 100, y: 200 }, data: { label: 'Evaluere tilbud' } },
  { id: 'ansk-forhandling', type: 'flowcrt', position: { x: 300, y: 200 }, data: { label: 'Forhandling' } },
  { id: 'ansk-bestilling', type: 'flowcrt', position: { x: 500, y: 200 }, data: { label: 'Bestilling' } },
  { id: 'ansk-oppfolging', type: 'flowcrt', position: { x: 700, y: 200 }, data: { label: 'Oppfølging' } },
  // ... flere noder
];
```

---

## Oppsummering av endringer

- **39 filer endret**, 3186+ linjer lagt til, 275 linjer slettet
- Redigerbare noder med dobbeltklikk og `window.prompt()`
- Slette-knapp som vises ved hover for editable nodes
- Multi-directional handles: 8 handles (top/bottom/left/right, target/source)
- LabelNode-komponent for tekst uten handles
- 5 nye ISO 9001-undersider med 71 nye noder totalt
- Oppgradert alle 9 eksisterende undersider til realistic prosessnoder
- Totalt 14 prosessområder i ISO 9001
- Flask API med PostgreSQL-integrasjon
- Register-funksjonalitet
- Massiv CSS-utvidelse med 659+ nye linjer
- Flow-wrapper høyde økt fra 500px til 750px
