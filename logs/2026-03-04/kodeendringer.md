# Kodeendringer - 2026-03-04

Dette dokumentet viser de faktiske kodeendringene som ble gjort 4. mars 2026 i FlowCRT-prosjektet.

**Diff-område:** `247ac8d..199bf09` (reorganisering + rollebasert redigering + flow-persistens)

**Merk:** To substantielle commits. Første commit reorganiserer mappestruktur og oppretter ny `StaticFlowPage`. Andre commit implementerer rollebasert redigering, API-persistens og handle-kontroll.

---

## Hovedendringer

### 1. Ny komponent: StaticFlowPage.jsx

#### `Frontend/src/pages/ISO-sider/9001-Undersider/Statiske/StaticFlowPage.jsx` (NY FIL)

```jsx
import { useState, useCallback, useEffect } from 'react';
import { ReactFlow, applyNodeChanges, applyEdgeChanges } from '@xyflow/react';
import '@xyflow/react/dist/style.css';
import { useNavigate } from 'react-router-dom';
import FlowCRTNode from '../../../../components/FlowCRTNode';
import LabelNode from '../../../../components/LabelNode';

const nodeTypes = {
  flowcrt: FlowCRTNode,
  label: LabelNode,
};

/**
 * Template for static ISO 9001 sub-pages with predefined nodes/edges.
 * Individual pages only need to pass pageTitle, initialNodes, and initialEdges.
 */
export default function StaticFlowPage({ pageTitle, initialNodes = [], initialEdges = [] }) {
  const [nodes, setNodes] = useState(initialNodes);
  const [edges, setEdges] = useState(initialEdges);
  const [rfInstance, setRfInstance] = useState(null);
  const navigate = useNavigate();

  useEffect(() => {
    if (!rfInstance) return;
    const handleResize = () => rfInstance.fitView({ padding: 0.2 });
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, [rfInstance]);

  const onNodesChange = useCallback(
    (changes) => setNodes((nds) => applyNodeChanges(changes, nds)),
    [],
  );
  const onEdgesChange = useCallback(
    (changes) => setEdges((eds) => applyEdgeChanges(changes, eds)),
    [],
  );

  // ... stilobjekter ...

  return (
    <div style={containerStyle}>
      <div style={headerStyle}>
        <h1 style={titleStyle}>{pageTitle}</h1>
        <button style={backButtonStyle} onClick={() => navigate('/iso9001')}>
          Til hovedprosess
        </button>
      </div>
      <div style={flowWrapperStyle}>
        <ReactFlow
          nodes={nodes}
          edges={edges}
          onNodesChange={onNodesChange}
          onEdgesChange={onEdgesChange}
          nodeTypes={nodeTypes}
          minZoom={0.3}
          proOptions={{ hideAttribution: true }}
          onInit={setRfInstance}
          fitView
          fitViewOptions={{ padding: 0.15 }}
        />
      </div>
    </div>
  );
}
```

**Forklaring:**
- Ingen redigeringsfunksjonalitet - `FlowCRTNode` vil motta `editable: false` fra node-dataene
- Ingen "Lagre"-knapp, ingen node-verktøylinje
- Identisk `fitView`-mønster som `FlowPageTemplate` for konsistent brukeropplevelse
- Props-grensesnittet er bevisst minimalt: bare `pageTitle`, `initialNodes`, `initialEdges`

---

### 2. StrategiOgStyring - predefinert node/edge-layout

#### `Frontend/src/pages/ISO-sider/9001-Undersider/Dynamiske/StrategiOgStyring.jsx` (NY FIL)

```jsx
import { MarkerType } from '@xyflow/react';
import FlowPageTemplate from './FlowPageTemplate';

const edgeStyle = { stroke: 'black', strokeWidth: 2 };
const edgeMarker = { type: MarkerType.ArrowClosed, color: 'black' };

const templateNodes = [
  // Styrende dokumenter (øverste rad)
  { id: 'n1', type: 'flowcrt', position: { x: 0,    y: 0 }, data: { label: 'Omfangsdokument',  description: 'Definerer virksomhetens omfang',      editable: true, allHandles: true }, draggable: true },
  { id: 'n2', type: 'flowcrt', position: { x: 350,  y: 0 }, data: { label: 'Kvalitetspolitikk', description: 'Overordnede kvalitetsprinsipper',      editable: true, allHandles: true }, draggable: true },
  { id: 'n3', type: 'flowcrt', position: { x: 700,  y: 0 }, data: { label: 'Kvalitetsmål',      description: 'Målbare mål for kvalitet',             editable: true, allHandles: true }, draggable: true },
  { id: 'n4', type: 'flowcrt', position: { x: 1050, y: 0 }, data: { label: 'Organisasjonskart', description: 'Roller, ansvar og rapporteringslinjer', editable: true, allHandles: true }, draggable: true },
  { id: 'n5', type: 'flowcrt', position: { x: 1400, y: 0 }, data: { label: 'Systembeskrivelse', description: 'Oversikt over styringssystemet',        editable: true, allHandles: true }, draggable: true },
  { id: 'n6', type: 'flowcrt', position: { x: 1750, y: 0 }, data: { label: 'Strategisk plan',   description: 'Langsiktige mål og tiltak',            editable: true, allHandles: true }, draggable: true },

  // 2x2 tom-grid (bunn venstre)
  { id: 'g1', type: 'flowcrt', position: { x: 0,   y: 400 }, data: { label: '', editable: true, allHandles: true }, draggable: true },
  { id: 'g2', type: 'flowcrt', position: { x: 350, y: 400 }, data: { label: '', editable: true, allHandles: true }, draggable: true },
  { id: 'g3', type: 'flowcrt', position: { x: 0,   y: 550 }, data: { label: '', editable: true, allHandles: true }, draggable: true },
  { id: 'g4', type: 'flowcrt', position: { x: 350, y: 550 }, data: { label: '', editable: true, allHandles: true }, draggable: true },

  // Kunder og leverandører (bunn høyre)
  { id: 'kunder',       type: 'flowcrt', position: { x: 1750, y: 400 }, data: { label: 'Kunder',       description: 'Mottakere av produkter og tjenester', editable: true, allHandles: true }, draggable: true },
  { id: 'leverandorer', type: 'flowcrt', position: { x: 1750, y: 550 }, data: { label: 'Leverandører', description: 'Eksterne leverandører og partnere',    editable: true, allHandles: true }, draggable: true },
];

const templateEdges = [
  { id: 'n1-n2', source: 'n1', target: 'n2', sourceHandle: 'right-source', targetHandle: 'left-target', type: 'straight', style: edgeStyle, markerEnd: edgeMarker },
  { id: 'n2-n3', source: 'n2', target: 'n3', sourceHandle: 'right-source', targetHandle: 'left-target', type: 'straight', style: edgeStyle, markerEnd: edgeMarker },
  { id: 'n3-n4', source: 'n3', target: 'n4', sourceHandle: 'right-source', targetHandle: 'left-target', type: 'straight', style: edgeStyle, markerEnd: edgeMarker },
  { id: 'n4-n5', source: 'n4', target: 'n5', sourceHandle: 'right-source', targetHandle: 'left-target', type: 'straight', style: edgeStyle, markerEnd: edgeMarker },
  { id: 'n5-n6', source: 'n5', target: 'n6', sourceHandle: 'right-source', targetHandle: 'left-target', type: 'straight', style: edgeStyle, markerEnd: edgeMarker },
];

export default function StrategiOgStyring() {
  return (
    <FlowPageTemplate
      pageTitle="Strategi og styring"
      pageSlug="strategi-og-styring"
      templateNodes={templateNodes}
      templateEdges={templateEdges}
    />
  );
}
```

**Forklaring:**
- Alle noder bruker `allHandles: true` slik at administratorer kan koble dem i alle retninger
- 4 tomme grid-noder (g1-g4) er tiltenkt fri bruk av administratorer
- `pageSlug="strategi-og-styring"` kobler siden mot API-endepunktet `POST/GET /flows/strategi-og-styring`

---

### 3. Rollebasert redigering i FlowPageTemplate

#### `Frontend/src/pages/ISO-sider/9001-Undersider/Dynamiske/FlowPageTemplate.jsx` (ENDRET)

```diff
+import { useAuth } from '../../../../auth/AuthContext';

-export default function FlowPageTemplate({ pageTitle, templateNodes, templateEdges }) {
+export default function FlowPageTemplate({ pageTitle, templateNodes, templateEdges, pageSlug }) {
+  const { user } = useAuth();
+  const canEdit = user?.role === 'FLOW_ADMIN' || user?.role === 'SYSTEM_ADMIN';

   const {
     nodes, edges, onNodesChange, onEdgesChange, onConnect,
-    onNodeClick, onNodeDoubleClick, addNodeInDirection,
+    onNodeClick, onNodeDoubleClick, addNodeInDirection, saveFlow,
     goBack, setRfInstance,
-  } = useFlowPage(pageTitle, templateNodes, templateEdges);
+  } = useFlowPage(pageTitle, templateNodes, templateEdges, canEdit, pageSlug);
```

```diff
+      {canEdit && pageSlug && (
+        <button
+          style={{ ...backButtonStyle, marginLeft: '10px', background: '#1a5c1a' }}
+          onClick={saveFlow}
+        >
+          Lagre
+        </button>
+      )}
-      <div style={toolbarStyle}>
+      {canEdit && <div style={toolbarStyle}>
         {/* ... node-retningsknapper ... */}
-      </div>
+      </div>}
```

**Forklaring:**
- Gjester og vanlige brukere som besøker siden ser kun diagrammet - ingen verktøylinje og ingen Lagre-knapp
- `canEdit`-flagget sendes ned til `useFlowPage` som setter `draggable: false` og `editable: false` på alle noder for ikke-administratorer
- Grønn "Lagre"-knapp vises bare dersom bruker har riktig rolle OG `pageSlug` er satt

---

### 4. API-persistens og canEdit i useFlowPage

#### `Frontend/src/layout/useFlowPage.js` (ENDRET)

```diff
+const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || 'http://localhost:8000';

-export function useFlowPage(pageTitle, templateNodes = null, templateEdges = null) {
+export function useFlowPage(pageTitle, templateNodes = null, templateEdges = null, canEdit = true, pageSlug = null) {

+  const applyCanEdit = (nds) => canEdit
+    ? nds
+    : nds.map((n) => ({ ...n, draggable: false, data: { ...n.data, editable: false } }));
+
+  const sourceNodes = templateNodes ?? defaultNodes;
-  const [nodes, setNodes] = useState(templateNodes ?? defaultNodes);
+  const [nodes, setNodes] = useState(applyCanEdit(sourceNodes));

+  // Load saved flow from API on mount
+  useEffect(() => {
+    if (!pageSlug) return;
+    fetch(`${API_BASE_URL}/flows/${pageSlug}`, { credentials: 'include' })
+      .then((r) => (r.ok ? r.json() : null))
+      .then((saved) => {
+        if (!saved?.nodes) return;
+        setNodes(applyCanEdit(saved.nodes));
+        if (saved.edges) setEdges(saved.edges);
+      })
+      .catch(() => {});
+  }, [pageSlug]);

+  // Save full flow to API
+  const saveFlow = useCallback(async () => {
+    if (!pageSlug) return;
+    await fetch(`${API_BASE_URL}/flows/${pageSlug}`, {
+      method: 'POST',
+      headers: { 'Content-Type': 'application/json' },
+      credentials: 'include',
+      body: JSON.stringify({ nodes, edges }),
+    });
+  }, [pageSlug, nodes, edges]);

   return {
     // ... eksisterende ...
+    saveFlow,
   };
```

**Forklaring:**
- `credentials: 'include'` sender session-cookies automatisk, slik at API-et vet hvem som er innlogget
- `applyCanEdit` kjøres både på initialtilstand og på data lastet fra API, slik at rollen alltid respekteres
- `saveFlow` sender `nodes` og `edges` som JSON direkte (ikke stringified) - backend forventer JSON-objekt

---

### 5. Handle-kontroll i FlowCRTNode

#### `Frontend/src/components/FlowCRTNode.jsx` (ENDRET)

```diff
-      ) : (
+      ) : data.showHandles === false ? null : (
         <>
-          <Handle type="target" position={Position.Left} id="left-target" />
-          <Handle type="source" position={Position.Right} id="right-source" />
+          {!data.hideTargetHandle && <Handle type="target" position={Position.Left} id="left-target" />}
+          {!data.hideSourceHandle && <Handle type="source" position={Position.Right} id="right-source" />}
         </>
       )}
```

**Forklaring:**
- Tre nivåer av handle-synlighet i standardmodus (ikke `allHandles`):
  1. `data.showHandles === false`: ingen handles i det hele tatt
  2. `data.hideTargetHandle`: skjuler kun venstre innkommende handle
  3. `data.hideSourceHandle`: skjuler kun høyre utgående handle
- Brukes på ISO9001-hovedsiden der første node (n1) ikke skal ha innkommende pil og siste (n7) ikke skal ha utgående

---

### 6. Handle-justering på ISO9001-hovedside

#### `Frontend/src/pages/ISO-sider/ISO9001.jsx` (ENDRET)

```diff
 // Node n1 (Salg og kontrakts-håndtering) - første node i rekken
   data: {
     label: 'Salg og kontrakts-håndtering',
     description: 'Tilbud, kontrakter og kundekrav',
-    slug: 'salg-og-kontrakts-haandtering'
+    slug: 'salg-og-kontrakts-haandtering',
+    hideTargetHandle: true       // <- ingen innkommende pil på første node
   },

 // Node n7 (Kunde tilfredshet) - siste node i rekken
   data: {
     label: 'Kunde tilfredshet',
     description: 'Feedback og kontinuerlig forbedring',
-    slug: 'kunde-tilfredshet'
+    slug: 'kunde-tilfredshet',
+    hideSourceHandle: true       // <- ingen utgående pil på siste node
   },
```

**Forklaring:**
- Hoved-ISO9001-siden viser prosessrekken fra venstre til høyre
- Første node skal kun ha utgående handle (ingen pil inn), siste node kun innkommende (ingen pil ut)
- Dette gir en visuelt tydelig start og slutt på prosessrekken uten at det krever spesialhåndtering i kantkoden

---

## Oppsummering av endringer

- **25 filer endret/opprettet** totalt fordelt på to commits
- Ny tolagsarkitektur: `Statiske/` vs `Dynamiske/` undermapper
- `StaticFlowPage`: gjenbrukbar visningskomponent uten redigeringsfunksjonalitet
- `FlowPageTemplate` + `useFlowPage`: kobler nå `useAuth`-rolle til `canEdit` og API-persistens
- `FlowCRTNode`: tre nivåer av handle-kontroll (`showHandles`, `hideTargetHandle`, `hideSourceHandle`)
- `StrategiOgStyring`: første underside med fullstendig predefinert node/edge-layout
- Neste steg: rulle ut `pageSlug` til øvrige dynamiske undersider og definere `page_flows`-tabellen i backend
