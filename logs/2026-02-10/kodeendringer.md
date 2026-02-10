# Kodeendringer - 10. februar 2026

## Oversikt
Uncommitted endringer fra branchen `9001-modifikasjoner` i FlowCRT-repoet. Endringene omfatter utvidelse av ISO 9001-siden med flere noder og edges, samt forbedringer i responsivitet, styling og pan-begrensning.

---

## Diff

```diff
diff --git a/Frontend/src/components/FlowCRTNode.jsx b/Frontend/src/components/FlowCRTNode.jsx
index bc9362e..d603dec 100644
--- a/Frontend/src/components/FlowCRTNode.jsx
+++ b/Frontend/src/components/FlowCRTNode.jsx
@@ -9,7 +9,13 @@ export default function FlowCRTNode({ data }) {
       <Handle type="target" position={Position.Left} id="left" />

       {/* Høyre (output) */}
-      <Handle type="source" position={Position.Right} id="right" />
+      <Handle
+        type="source"
+        position={Position.Right}
+        id="right"
+      />
+
+
     </div>
   );
 }
```

```diff
diff --git a/Frontend/src/pages/ISO-sider/ISO9001.jsx b/Frontend/src/pages/ISO-sider/ISO9001.jsx
index 1e1f415..fa096b0 100644
--- a/Frontend/src/pages/ISO-sider/ISO9001.jsx
+++ b/Frontend/src/pages/ISO-sider/ISO9001.jsx
@@ -1,4 +1,4 @@
-import { useState, useCallback } from 'react';
+import { useState, useCallback, useEffect } from 'react';
 import { ReactFlow, applyNodeChanges, applyEdgeChanges, addEdge, MarkerType } from '@xyflow/react';
 import '@xyflow/react/dist/style.css';
 import FlowCRTNode from '../../components/FlowCRTNode';
@@ -14,20 +14,67 @@ const initialNodes = [
     id: 'n1',
     type: 'flowcrt',
     position: { x: 0, y: 0 },
-    data: { label: 'Node 1' },
+    data: { label: 'Salg og kontrakts-håndtering' },
+    draggable: false,
   },
   {
     id: 'n2',
     type: 'flowcrt',
-    position: { x: 250, y: 100 },
-    data: { label: 'Node 2' },
+    position: { x: 350, y: 0 },
+    data: { label: 'Utforming og utvikling' },
+    draggable: false,
   },
   {
     id: 'n3',
     type: 'flowcrt',
-    position: { x: 500, y: 200 },
-    data: { label: 'Node 3' },
+    position: { x: 700, y: 0 },
+    data: { label: 'Anskaffelse' },
+    draggable: false,
   },
+  {
+    id: 'n4',
+    type: 'flowcrt',
+    position: { x: 1050, y: 0 },
+    data: { label: 'Produksjon og tjeneste leveranse' },
+    draggable: false,
+  },
+  {
+    id: 'n5',
+    type: 'flowcrt',
+    position: { x: 1400, y: 0 },
+    data: { label: 'Test og frigivelse' },
+    draggable: false,
+  },
+  {
+    id: 'n6',
+    type: 'flowcrt',
+    position: { x: 1750, y: 0 },
+    data: { label: 'Leveranse' },
+    draggable: false,
+  },
+  {
+    id: 'n7',
+    type: 'flowcrt',
+    position: { x: 2100, y: 0 },
+    data: { label: 'Kunde tilfredshet' },
+    draggable: false,
+  },
+
+  {
+    id: 'n8',
+    type: 'flowcrt',
+    position: { x: 1900, y: -350 },
+    data: { label: 'Visjon' },
+    draggable: false,
+  },
+
+  {
+    id: 'n9',
+    type: 'flowcrt',
+    position: { x: 1900, y: 400 },
+    data: { label: 'Kunder' },
+    draggable: false,
+  }
 ];

 const initialEdges = [
@@ -35,17 +82,93 @@ const initialEdges = [
     id: 'n1-n2',
     source: 'n1',
     target: 'n2',
-    type: 'step',
+    type: 'straight',
+    style: { stroke: 'black', strokeWidth: 2 },
+
+    markerEnd:
+    {
+      type: MarkerType.ArrowClosed,
+      color:'black',
+
+    },
+  },
+  {
+    id: 'n2-n3',
+    source: 'n2',
+    target: 'n3',
+    type: 'straight',
+    style: { stroke: 'black', strokeWidth: 2 },
     markerEnd:
     {
       type: MarkerType.ArrowClosed,
+      color:'black',
+
+    },
+  },
+  {
+    id: 'n3-n4',
+    source: 'n3',
+    target: 'n4',
+    type: 'straight',
+    style: { stroke: 'black', strokeWidth: 2 },
+    markerEnd:
+    {
+      type: MarkerType.ArrowClosed,
+      color:'black',
+
     },
   },
+  {id: 'n4-n5',
+    source: 'n4',
+    target: 'n5',
+    type: 'straight',
+    style: { stroke: 'black', strokeWidth: 2 },
+    markerEnd:
+    {
+      type: MarkerType.ArrowClosed,
+      color:'black',
+
+    },
+  },
+  {
+    id: 'n5-n6',
+    source: 'n5',
+    target: 'n6',
+    type: 'straight',
+    style: { stroke: 'black', strokeWidth: 2 },
+    markerEnd:
+    {
+      type: MarkerType.ArrowClosed,
+      color:'black',
+
+    },
+  },
+  {
+    id: 'n6-n7',
+    source: 'n6',
+    target: 'n7',
+    type: 'straight',
+    style: { stroke: 'black', strokeWidth: 2 },
+    markerEnd:
+    {
+      type: MarkerType.ArrowClosed,
+      color:'black',
+    },
+  },
+
 ];

 export default function App() {
   const [nodes, setNodes] = useState(initialNodes);
   const [edges, setEdges] = useState(initialEdges);
+  const [rfInstance, setRfInstance] = useState(null);
+
+  useEffect(() => {
+    if (!rfInstance) return;
+    const handleResize = () => rfInstance.fitView({ padding: 0.2 });
+    window.addEventListener('resize', handleResize);
+    return () => window.removeEventListener('resize', handleResize);
+  }, [rfInstance]);

   const onNodesChange = useCallback(
     (changes) => setNodes((nodesSnapshot) => applyNodeChanges(changes, nodesSnapshot)),
@@ -72,12 +195,15 @@ export default function App() {
           onEdgesChange={onEdgesChange}
           onConnect={onConnect}
           nodeTypes={nodeTypes}
-          /*nodeExtent={[
-            [0, 0],
-            [1000, 600],
-          ]}*/
+          minZoom={0.3}
+          translateExtent={[
+            [-200, -600],
+            [2600, 650],
+          ]}
            proOptions={{ hideAttribution: true }}
+          onInit={setRfInstance}
           fitView
+          fitViewOptions={{ padding: 0.2 }}
         />
       </div>
     </div>
```

```diff
diff --git a/Frontend/src/styles.css b/Frontend/src/styles.css
index 1ff957a..a42a964 100644
--- a/Frontend/src/styles.css
+++ b/Frontend/src/styles.css
@@ -137,8 +137,10 @@ button:hover {
   padding: 16px 24px;
   border-radius: 12px;
   border: 2px solid #014d01;
-  min-width: 160px;
+  width: 200px;
+  height: 50px;
   text-align: center;
+  font-size: 20px;
   font-weight: 500;
   box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
 }
@@ -167,7 +169,7 @@ button:hover {
 }

 .flow-wrapper {
-  height: 600px;
+  height: calc(100vh - 120px);
   border: 3px solid #002d00;
   border-radius: 16px;
   background: #c9c9c6;
```

---

## Oppsummering av endringer

### Filer endret:
1. **Frontend/src/components/FlowCRTNode.jsx** - Formatering av Handle-komponent
2. **Frontend/src/pages/ISO-sider/ISO9001.jsx** - Lagt til 6 nye noder (n4-n9) med beskrivende labels, oppdatert edges med styling, implementert resize-lytter med useEffect og rfInstance, lagt til translateExtent for pan-begrensning, endret minZoom til 0.3, fjernet utkommentert kode
3. **Frontend/src/styles.css** - Oppdatert node-styling med fast bredde/høyde og større font, samt dynamisk høyde på flow-wrapper
