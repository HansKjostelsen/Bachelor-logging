# Kodeendringer - 2026-02-05

Denne filen inneholder den komplette diff-en for commit `bf59251` ("Importert Flow, og endret ISO sider").

**Endrede filer:**
- `Frontend/index.html`
- `Frontend/package.json`
- `Frontend/src/App.jsx`
- `Frontend/src/components/FlowCRTNode.jsx` (ny fil)
- `Frontend/src/pages/ISO-sider/ISO9001.jsx` (ny fil)
- `Frontend/src/pages/ISO9001.jsx` (slettet)
- `Frontend/src/styles.css`

**Binærfiler og package-lock.json er ekskludert fra denne visningen.**

---

## Diff fra forrige commit (940ebe8..bf59251)

```diff
diff --git a/Frontend/index.html b/Frontend/index.html
index 6df2b85..ee279c7 100644
--- a/Frontend/index.html
+++ b/Frontend/index.html
@@ -2,7 +2,7 @@
 <html lang="en">
   <head>
     <meta charset="UTF-8" />
-    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
+    <link rel="icon" type="image/svg+xml" href="/FlowCRT_1-1_mindre.jpg" />
     <meta name="viewport" content="width=device-width, initial-scale=1.0" />
     <title>flowcrt-nettside</title>
     <link rel="stylesheet" href="/src/styles.css" />
```

```diff
diff --git a/Frontend/package.json b/Frontend/package.json
index 45b20af..47cc780 100644
--- a/Frontend/package.json
+++ b/Frontend/package.json
@@ -10,6 +10,7 @@
     "preview": "vite preview"
   },
   "dependencies": {
+    "@xyflow/react": "^12.10.0",
     "react": "^19.2.0",
     "react-dom": "^19.2.0",
     "react-router-dom": "^7.13.0"
```

```diff
diff --git a/Frontend/src/App.jsx b/Frontend/src/App.jsx
index 869db20..a9099f6 100644
--- a/Frontend/src/App.jsx
+++ b/Frontend/src/App.jsx
@@ -5,9 +5,9 @@ import Home from "./pages/Home";
 import FAQ from "./pages/FAQ";
 import About from "./pages/about";
 import Contact from "./pages/Contact";
-import ISO9001 from "./pages/ISO9001";
-import ISO14001 from "./pages/ISO14001";
-import ISO27001 from "./pages/ISO27001";
+import ISO9001 from "./pages/ISO-sider/ISO9001";
+import ISO14001 from "./pages/ISO-sider/ISO14001";
+import ISO27001 from "./pages/ISO-sider/ISO27001";

 import { BrowserRouter, Routes, Route } from "react-router-dom";
```

```diff
diff --git a/Frontend/src/components/FlowCRTNode.jsx b/Frontend/src/components/FlowCRTNode.jsx
new file mode 100644
index 0000000..bc9362e
--- /dev/null
+++ b/Frontend/src/components/FlowCRTNode.jsx
@@ -0,0 +1,15 @@
+import { Handle, Position } from '@xyflow/react';
+
+export default function FlowCRTNode({ data }) {
+  return (
+    <div className="flowcrt-node">
+      <div className="flowcrt-title">{data.label}</div>
+
+      {/* Venstre (input) */}
+      <Handle type="target" position={Position.Left} id="left" />
+
+      {/* Høyre (output) */}
+      <Handle type="source" position={Position.Right} id="right" />
+    </div>
+  );
+}
```

```diff
diff --git a/Frontend/src/pages/ISO14001.jsx b/Frontend/src/pages/ISO-sider/ISO14001.jsx
similarity index 100%
rename from Frontend/src/pages/ISO14001.jsx
rename to Frontend/src/pages/ISO-sider/ISO14001.jsx
```

```diff
diff --git a/Frontend/src/pages/ISO27001.jsx b/Frontend/src/pages/ISO-sider/ISO27001.jsx
similarity index 100%
rename from Frontend/src/pages/ISO27001.jsx
rename to Frontend/src/pages/ISO-sider/ISO27001.jsx
```

```diff
diff --git a/Frontend/src/pages/ISO-sider/ISO9001.jsx b/Frontend/src/pages/ISO-sider/ISO9001.jsx
new file mode 100644
index 0000000..1e1f415
--- /dev/null
+++ b/Frontend/src/pages/ISO-sider/ISO9001.jsx
@@ -0,0 +1,85 @@
+import { useState, useCallback } from 'react';
+import { ReactFlow, applyNodeChanges, applyEdgeChanges, addEdge, MarkerType } from '@xyflow/react';
+import '@xyflow/react/dist/style.css';
+import FlowCRTNode from '../../components/FlowCRTNode';
+
+const nodeTypes = {
+  flowcrt: FlowCRTNode,
+};
+
+
+const initialNodes = [
+
+  {
+    id: 'n1',
+    type: 'flowcrt',
+    position: { x: 0, y: 0 },
+    data: { label: 'Node 1' },
+  },
+  {
+    id: 'n2',
+    type: 'flowcrt',
+    position: { x: 250, y: 100 },
+    data: { label: 'Node 2' },
+  },
+  {
+    id: 'n3',
+    type: 'flowcrt',
+    position: { x: 500, y: 200 },
+    data: { label: 'Node 3' },
+  },
+];
+
+const initialEdges = [
+  {
+    id: 'n1-n2',
+    source: 'n1',
+    target: 'n2',
+    type: 'step',
+    markerEnd:
+    {
+      type: MarkerType.ArrowClosed,
+    },
+  },
+];
+
+export default function App() {
+  const [nodes, setNodes] = useState(initialNodes);
+  const [edges, setEdges] = useState(initialEdges);
+
+  const onNodesChange = useCallback(
+    (changes) => setNodes((nodesSnapshot) => applyNodeChanges(changes, nodesSnapshot)),
+    [],
+  );
+  const onEdgesChange = useCallback(
+    (changes) => setEdges((edgesSnapshot) => applyEdgeChanges(changes, edgesSnapshot)),
+    [],
+  );
+  const onConnect = useCallback(
+    (params) => setEdges((edgesSnapshot) => addEdge({ ...params, type: 'step', markerEnd: { type: MarkerType.ArrowClosed } }, edgesSnapshot)),
+    [],
+  );
+
+  return (
+    <div className="ISO-side">
+      <h1 className="ISO-tittel">ISO 9001 - Prosessoversikt</h1>
+
+      <div className="flow-wrapper">
+       <ReactFlow
+          nodes={nodes}
+          edges={edges}
+          onNodesChange={onNodesChange}
+          onEdgesChange={onEdgesChange}
+          onConnect={onConnect}
+          nodeTypes={nodeTypes}
+          /*nodeExtent={[
+            [0, 0],
+            [1000, 600],
+          ]}*/
+           proOptions={{ hideAttribution: true }}
+          fitView
+        />
+      </div>
+    </div>
+  );
+}
```

```diff
diff --git a/Frontend/src/pages/ISO9001.jsx b/Frontend/src/pages/ISO9001.jsx
deleted file mode 100644
index a2fe1ef..0000000
--- a/Frontend/src/pages/ISO9001.jsx
+++ /dev/null
@@ -1,12 +0,0 @@
-import React from "react";
-
-function ISO9001() {
-  return (
-    <main>
-      <h2>ISO 9001</h2>
-      <p>Dette er siden for ISO 9001 sertifisering.</p>
-    </main>
-  );
-}
-
-export default ISO9001;
```

```diff
diff --git a/Frontend/src/styles.css b/Frontend/src/styles.css
index 7398952..1ff957a 100644
--- a/Frontend/src/styles.css
+++ b/Frontend/src/styles.css
@@ -1,7 +1,7 @@
 body {
   margin: 0;
   font-family: 'Segoe UI', Arial, sans-serif;
-  background: #f6f8fa;
+  background: #ffff;
   color: #222;
   min-height: 100vh;
   display: flex;
@@ -24,15 +24,16 @@ footer, .footer {
 }

 header, .header {
-  background: #123223; /* Navy */
+  background: #013220; /* Navy */
   color: #fff;
-  padding: 0 0 1rem 0;
+  padding: 1rem 1rem 1rem 0;
   margin-top: 0;
+
 }

 footer, .footer {
-  background: #123223; /* Navy */
-  color: #fff;
+  background: #013220; /* Navy */
+  color: #ffffff;
   padding: 1rem 0;
   margin-bottom: 0;
   text-align: center;
@@ -129,3 +130,46 @@ button {
 button:hover {
   background: #123223;
 }
+
+.flowcrt-node {
+  background: #013220;
+  color: white;
+  padding: 16px 24px;
+  border-radius: 12px;
+  border: 2px solid #014d01;
+  min-width: 160px;
+  text-align: center;
+  font-weight: 500;
+  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
+}
+
+.flowcrt-node:hover {
+  background: #DDB771;
+  color: #000000;
+}
+
+/* Handles */
+.react-flow__handle {
+  width: 10px;
+  height: 10px;
+  background: white;
+  border: 2px solid #002d00;
+}
+
+.ISO-side {
+  padding: 40px;
+}
+
+.ISO-tittel {
+  font-size: 28px;
+  margin-bottom: 20px;
+  color: #002d00;
+}
+
+.flow-wrapper {
+  height: 600px;
+  border: 3px solid #002d00;
+  border-radius: 16px;
+  background: #c9c9c6;
+  overflow: hidden;
+}
```
