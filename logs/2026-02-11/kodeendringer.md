# Kodeendringer - 2026-02-11

Dette dokumentet viser de faktiske kodeendringene som ble gjort 11. februar 2026 i FlowCRT-prosjektet.

**Diff-område:** `5bfacbd..82c75ff` (fra første til siste commit denne dagen)

**Merk:** Dette var en stor dag med 40 filer endret, 1331+ linjer lagt til. Dette dokumentet viser de viktigste endringene.

---

## Hovedendringer

### 1. FlowCRTNode - Navigasjon og hover-funksjonalitet

#### Frontend/src/components/FlowCRTNode.jsx

```diff
+import { useState } from 'react';
 import { Handle, Position } from '@xyflow/react';
+import { useNavigate } from 'react-router-dom';

 export default function FlowCRTNode({ data }) {
+  const [isHovered, setIsHovered] = useState(false);
+  const navigate = useNavigate();
+
+  const handleClick = () => {
+    if (data.slug) {
+      navigate(`/iso9001/${data.slug}`);
+    }
+  };
+
   return (
-    <div className="flowcrt-node">
-      <div className="flowcrt-title">{data.label}</div>
+    <div
+      className="flowcrt-node"
+      onMouseEnter={() => setIsHovered(true)}
+      onMouseLeave={() => setIsHovered(false)}
+      onClick={handleClick}
+    >
+      <div className="flowcrt-label">{data.label}</div>
+
+      {isHovered && data.description && (
+        <div className="flowcrt-description">{data.description}</div>
+      )}

-      {/* Venstre (input) */}
       <Handle type="target" position={Position.Left} id="left" />
-
-      {/* Høyre (output) */}
       <Handle type="source" position={Position.Right} id="right" />
     </div>
   );
 }
```

**Forklaring:**
- Lagt til `useState` for å tracke hover-tilstand
- Lagt til `useNavigate` for routing
- `handleClick` funksjon navigerer til underside basert på slug
- Hover-effekt som viser description når musen er over noden

---

### 2. ISO9001 - Responsiv fitView og utvidede noder

#### Frontend/src/pages/ISO-sider/ISO9001.jsx

```diff
-import { useState, useCallback } from 'react';
+import { useState, useCallback, useEffect } from 'react';
 import { ReactFlow, applyNodeChanges, applyEdgeChanges, addEdge, MarkerType } from '@xyflow/react';
 import '@xyflow/react/dist/style.css';
 import FlowCRTNode from '../../components/FlowCRTNode';

 const initialNodes = [
   {
     id: 'n1',
     type: 'flowcrt',
     position: { x: 0, y: 0 },
-    data: { label: 'Node 1' },
+    data: {
+      label: 'Salg og kontrakts-håndtering',
+      description: 'Tilbud, kontrakter og kundekrav',
+      slug: 'salg-og-kontrakts-haandtering'
+    },
+    draggable: false,
   },
   {
     id: 'n2',
     type: 'flowcrt',
-    position: { x: 250, y: 100 },
-    data: { label: 'Node 2' },
+    position: { x: 350, y: 0 },
+    data: {
+      label: 'Utforming og utvikling',
+      description: 'Design, prototyping og dokumentasjon',
+      slug: 'utforming-og-utvikling'
+    },
+    draggable: false,
   },
+  // ... 7 flere noder lagt til (n3-n9)
 ];
```

**React Flow konfigurasjonsendringer:**

```diff
 export default function App() {
   const [nodes, setNodes] = useState(initialNodes);
   const [edges, setEdges] = useState(initialEdges);
+  const [rfInstance, setRfInstance] = useState(null);
+
+  useEffect(() => {
+    if (!rfInstance) return;
+    const handleResize = () => rfInstance.fitView({ padding: 0.1 });
+    window.addEventListener('resize', handleResize);
+    return () => window.removeEventListener('resize', handleResize);
+  }, [rfInstance]);

   return (
     <div className="ISO-side">
       <h1 className="ISO-tittel">ISO 9001 - Prosessoversikt</h1>
       <div className="flow-wrapper">
         <ReactFlow
           nodes={nodes}
           edges={edges}
           onNodesChange={onNodesChange}
           onEdgesChange={onEdgesChange}
           onConnect={onConnect}
           nodeTypes={nodeTypes}
-          minZoom={0.7}
-          maxZoom={0.7}
+          minZoom={0.3}
+          onInit={setRfInstance}
+          fitViewOptions={{ padding: 0.1 }}
+          translateExtent={[[-200, -600], [2600, 650]]}
           proOptions={{ hideAttribution: true }}
           fitView
         />
       </div>
     </div>
   );
 }
```

**Forklaring:**
- Lagt til `useEffect` med window resize-lytter for responsiv skalering
- `rfInstance` state lagrer React Flow-instansen
- Fjernet låst zoom (0.7) og satt `minZoom` til 0.3
- `translateExtent` begrenser pan-området
- `fitViewOptions` med 10% padding

---

### 3. Nye undersider (9 stk)

Alle 9 undersider følger samme struktur. Eksempel: **SalgOgKontraktshaandtering.jsx**

```jsx
import { useState, useCallback } from 'react';
import { ReactFlow, applyNodeChanges, applyEdgeChanges, addEdge, MarkerType } from '@xyflow/react';
import '@xyflow/react/dist/style.css';
import FlowCRTNode from '../../../components/FlowCRTNode';
import { useNavigate } from 'react-router-dom';

const nodeTypes = {
  flowcrt: FlowCRTNode,
};

const initialNodes = [
  {
    id: 'sale1',
    type: 'flowcrt',
    position: { x: 0, y: 0 },
    data: { label: 'Salgsforespørsel' },
    draggable: false,
  },
  {
    id: 'sale2',
    type: 'flowcrt',
    position: { x: 200, y: 0 },
    data: { label: 'Tilbudshåndtering' },
    draggable: false,
  },
  // ... flere noder
];

const initialEdges = [
  {
    id: 'sale1-sale2',
    source: 'sale1',
    target: 'sale2',
    type: 'straight',
    style: { stroke: 'black', strokeWidth: 2 },
    markerEnd: { type: MarkerType.ArrowClosed, color: 'black' },
  },
  // ... flere edges
];

export default function SalgOgKontraktshaandtering() {
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
      <h1 className="ISO-tittel">Salg og kontraktshåndtering</h1>

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

**De 9 undersidene:**
1. `SalgOgKontraktshaandtering.jsx`
2. `UtformingOgUtvikling.jsx`
3. `Anskaffelse.jsx`
4. `ProduksjonOgTjenesteLeveranse.jsx`
5. `TestOgFrigivelse.jsx`
6. `Leveranse.jsx`
7. `KundeTilfredshet.jsx`
8. `Visjon.jsx`
9. `Kunder.jsx`

Hver har 3-4 placeholder-noder og en tilbake-knapp.

---

### 4. App.jsx - Routing

#### Frontend/src/App.jsx

```diff
+import SalgOgKontraktshaandtering from "./pages/ISO-sider/9001-Undersider/SalgOgKontraktshaandtering";
+import UtformingOgUtvikling from "./pages/ISO-sider/9001-Undersider/UtformingOgUtvikling";
+import Anskaffelse from "./pages/ISO-sider/9001-Undersider/Anskaffelse";
+import ProduksjonOgTjenesteLeveranse from "./pages/ISO-sider/9001-Undersider/ProduksjonOgTjenesteLeveranse";
+import TestOgFrigivelse from "./pages/ISO-sider/9001-Undersider/TestOgFrigivelse";
+import Leveranse from "./pages/ISO-sider/9001-Undersider/Leveranse";
+import KundeTilfredshet from "./pages/ISO-sider/9001-Undersider/KundeTilfredshet";
+import Visjon from "./pages/ISO-sider/9001-Undersider/Visjon";
+import Kunder from "./pages/ISO-sider/9001-Undersider/Kunder";
+import Login from "./pages/login";

 function App() {
   return (
     <BrowserRouter>
       <div className="App">
         <Header />
         <Routes>
           <Route path="/" element={<Home />} />
           <Route path="/about" element={<About />} />
           <Route path="/faq" element={<FAQ />} />
           <Route path="/contact" element={<Contact />} />
           <Route path="/iso9001" element={<ISO9001 />} />
+          <Route path="/iso9001/salg-og-kontrakts-haandtering" element={<SalgOgKontraktshaandtering />} />
+          <Route path="/iso9001/utforming-og-utvikling" element={<UtformingOgUtvikling />} />
+          <Route path="/iso9001/anskaffelse" element={<Anskaffelse />} />
+          <Route path="/iso9001/produksjon-og-tjeneste-leveranse" element={<ProduksjonOgTjenesteLeveranse />} />
+          <Route path="/iso9001/test-og-frigivelse" element={<TestOgFrigivelse />} />
+          <Route path="/iso9001/leveranse" element={<Leveranse />} />
+          <Route path="/iso9001/kunde-tilfredshet" element={<KundeTilfredshet />} />
+          <Route path="/iso9001/visjon" element={<Visjon />} />
+          <Route path="/iso9001/kunder" element={<Kunder />} />
+          <Route path="/login" element={<Login />} />
           <Route path="/iso14001" element={<ISO14001 />} />
           <Route path="/iso27001" element={<ISO27001 />} />
         </Routes>
         <Footer />
       </div>
     </BrowserRouter>
   );
 }
```

**Forklaring:**
- 10 nye imports (9 undersider + Login)
- 10 nye ruter med slug-baserte paths

---

### 5. CSS-forbedringer

#### Frontend/src/styles.css (utvalgte endringer)

```diff
+.flowcrt-node {
+  cursor: pointer;
+  transition: transform 0.2s ease, box-shadow 0.2s ease;
+  height: 70px; /* Økt fra 50px */
+  display: flex;
+  flex-direction: column;
+  justify-content: center;
+}

+.flowcrt-node:hover {
+  transform: scale(1.05);
+  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.4);
+}

+.flowcrt-description {
+  font-size: 12px;
+  text-align: center;
+  font-weight: normal;
+  margin-top: 4px;
+}

+.back-button {
+  background: #4CAF50;
+  color: white;
+  border: none;
+  padding: 10px 20px;
+  border-radius: 5px;
+  cursor: pointer;
+  font-size: 16px;
+  margin-bottom: 20px;
+  transition: background 0.3s ease, transform 0.2s ease;
+}

+.back-button:hover {
+  background: #FFD700;
+  color: black;
+  transform: translateY(-2px);
+}
```

**Forklaring:**
- Node-høyde økt til 70px
- Pointer cursor og hover-skalering (scale 1.05)
- Description-styling for hover-tekst
- Back-button med grønn bakgrunn og gull hover-effekt

---

## Docker-konfigurasjon

Flere Docker-relaterte filer ble lagt til:

### API/Dockerfile

```dockerfile
FROM python:3.13-alpine
WORKDIR /app
COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt
COPY . .
CMD ["python3", "-m", "flask", "run", "--host=0.0.0.0", "--port=1337"]
EXPOSE 1337
```

### Frontend/Dockerfile

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
EXPOSE 5173
CMD ["npm", "run", "dev", "--", "--host"]
```

### Database/compose.yaml

```yaml
services:
  postgres:
    container_name: iso-postgres
    image: postgres:16-alpine
    ports:
      - 5432:5432
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: isobase
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./migration:/docker-entrypoint-initdb.d
volumes:
  postgres_data:
```

---

## Login-side

### Frontend/src/pages/login.jsx

```jsx
import React from "react";

function Login() {
  return (
    <main>
      <h2>Logg inn</h2>
      <form>
        <input type="text" placeholder="Brukernavn" />
        <input type="password" placeholder="Passord" />
        <button type="submit">Logg inn</button>
      </form>
    </main>
  );
}

export default Login;
```

---

## Oppsummering av endringer

- **40 filer endret**, 1331+ linjer lagt til, 21 linjer slettet
- Responsiv fitView med window resize-lytter
- 9 nye ISO 9001-undersider med React Flow-visualiseringer
- Klikkbare noder med navigasjon via slug-system
- Hover-funksjonalitet som bytter mellom label og description
- CSS-forbedringer: hover-effekter, pointer cursor, back-button styling
- Docker-konfigurasjon for hele stacken
- Login-side lagt til
