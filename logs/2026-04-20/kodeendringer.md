# Kodeendringer - 2026-04-20

Dette dokumentet viser de faktiske kodeendringene som ble gjort 20. april 2026 i FlowCRT-prosjektet.

**Diff-område:** `0bbc3444..80244799` (fra insertNodeOnEdge-commit til react-helmet-async-commit)

**Merk:** To commits denne dagen. Fokus på innsetting av noder mellom eksisterende noder og hasUnsavedChanges.

---

## Hovedendringer

### 1. insertNodeOnEdge og hasUnsavedChanges

#### Frontend/src/auth/useFlowPage.js (ENDRET)

```diff
   const [selectedNode, setSelectedNode] = useState(null);
+  const [selectedEdge, setSelectedEdge] = useState(null);
+  const [hasUnsavedChanges, setHasUnsavedChanges] = useState(false);
```

```diff
   const onNodesChange = useCallback(
-    (changes) => setNodes((nds) => applyNodeChanges(changes, nds)),
+    (changes) => {
+      const meaningful = changes.some((c) => c.type !== 'select' && c.type !== 'dimensions');
+      if (meaningful) setHasUnsavedChanges(true);
+      setNodes((nds) => applyNodeChanges(changes, nds));
+    },
     [],
   );

   const onEdgesChange = useCallback(
-    (changes) => setEdges((eds) => applyEdgeChanges(changes, eds)),
+    (changes) => {
+      setHasUnsavedChanges(true);
+      setEdges((eds) => applyEdgeChanges(changes, eds));
+    },
     [],
   );

+  const onEdgeClick = useCallback((event, edge) => {
+    setSelectedEdge(edge);
+    setSelectedNode(null);
+  }, []);
+
+  const onPaneClick = useCallback(() => {
+    setSelectedNode(null);
+    setSelectedEdge(null);
+  }, []);
```

```diff
+  const insertNodeOnEdge = useCallback(() => {
+    if (!selectedEdge) return;
+
+    const sourceNode = nodes.find((n) => n.id === selectedEdge.source);
+    const targetNode = nodes.find((n) => n.id === selectedEdge.target);
+    if (!sourceNode || !targetNode) return;
+
+    // Plasser ny node i midtpunktet
+    const newX = (sourceNode.position.x + targetNode.position.x) / 2;
+    const newY = (sourceNode.position.y + targetNode.position.y) / 2;
+
+    // Forskyv target med halve avstanden
+    const shiftX = (targetNode.position.x - sourceNode.position.x) / 2;
+    const shiftY = (targetNode.position.y - sourceNode.position.y) / 2;
+
+    // Bestem handles basert på original edge
+    const oppositeHandle = {
+      'right-source': 'left-target',
+      'left-source': 'right-target',
+      'bottom-source': 'top-target',
+      'top-source': 'bottom-target',
+    };
+    const incomingTarget = oppositeHandle[selectedEdge.sourceHandle] ?? selectedEdge.targetHandle;
+
+    const newNodeId = `user-${nodeIdCounter}`;
+    const newNode = {
+      id: newNodeId,
+      type: 'flowcrt',
+      position: { x: newX, y: newY },
+      data: {
+        label: 'Ny prosess',
+        description: 'Dobbeltklikk for å endre navn',
+        editable: true,
+        allHandles: true,
+        onDelete: deleteNode,
+        onLabelChange: updateNodeLabel,
+      },
+      draggable: true,
+    };
+
+    const edgeStyle = { stroke: 'black', strokeWidth: 2 };
+    const edgeMarker = { type: MarkerType.ArrowClosed, color: 'black' };
+
+    // Erstatt gammel edge med to nye
+    const edgeToNew = {
+      id: `e-${selectedEdge.source}-${newNodeId}`,
+      source: selectedEdge.source,
+      target: newNodeId,
+      sourceHandle: selectedEdge.sourceHandle,
+      targetHandle: incomingTarget,
+      type: 'straight',
+      style: edgeStyle,
+      markerEnd: edgeMarker,
+    };
+
+    const edgeFromNew = {
+      id: `e-${newNodeId}-${selectedEdge.target}`,
+      source: newNodeId,
+      target: selectedEdge.target,
+      sourceHandle: selectedEdge.sourceHandle,
+      targetHandle: selectedEdge.targetHandle,
+      type: 'straight',
+      style: edgeStyle,
+      markerEnd: edgeMarker,
+    };
+
+    setNodes((nds) => [
+      ...nds.map((n) =>
+        n.id === targetNode.id
+          ? { ...n, position: { x: n.position.x + shiftX, y: n.position.y + shiftY } }
+          : n
+      ),
+      newNode,
+    ]);
+    setEdges((eds) => [
+      ...eds.filter((e) => e.id !== selectedEdge.id),
+      edgeToNew,
+      edgeFromNew,
+    ]);
+    setNodeIdCounter((c) => c + 1);
+    setSelectedEdge(null);
+    setSelectedNode(newNode);
+    setHasUnsavedChanges(true);
+  }, [selectedEdge, nodes, nodeIdCounter, deleteNode, updateNodeLabel]);
```

```diff
     // Eksporterte verdier
     nodes,
     edges,
     selectedNode,
+    selectedEdge,
+    hasUnsavedChanges,

     onNodesChange,
     onEdgesChange,
     onNodeClick,
     onNodeDoubleClick,
+    onEdgeClick,
+    onPaneClick,
     addNodeInDirection,
+    insertNodeOnEdge,
     saveFlow,
     goBack,
```

**Forklaring:**
- `insertNodeOnEdge` beregner midtpunktet mellom source og target for å plassere den nye noden, og forskyver target-noden med halve den opprinnelige avstanden slik at det blir jevnt gap
- `oppositeHandle`-mappingen sikrer at den nye nodens innkommende handle matcher den utgående handle-retningen fra source
- `hasUnsavedChanges` filtrerer bort `select`- og `dimensions`-endringer i `onNodesChange` for å unngå falske positive — kun faktiske posisjons- og strukturendringer trigger ulagret-tilstand
- `onEdgeClick` og `onPaneClick` muliggjør edge-seleksjon og deseleksjon, og er gjensidig eksklusiv med node-seleksjon
- `saveFlow` setter `hasUnsavedChanges` tilbake til `false` etter vellykket lagring

---

### 2. FlowPageTemplate oppdatert med nye callbacks

#### Frontend/src/pages/9001-Undersider/Dynamiske/FlowPageTemplate.jsx (ENDRET)

```diff
   const {
     nodes, edges, selectedNode,
+    selectedEdge, hasUnsavedChanges,
     onNodesChange, onEdgesChange,
-    onNodeClick, onNodeDoubleClick,
-    addNodeInDirection,
+    onNodeClick, onNodeDoubleClick,
+    onEdgeClick, onPaneClick,
+    addNodeInDirection, insertNodeOnEdge,
     saveFlow, goBack,
   } = useFlowPage(pageTitle, templateNodes, templateEdges);
```

```diff
   <ReactFlow
     nodes={nodes}
     edges={edges}
     onNodesChange={onNodesChange}
     onEdgesChange={onEdgesChange}
     onNodeClick={onNodeClick}
     onNodeDoubleClick={onNodeDoubleClick}
+    onEdgeClick={onEdgeClick}
+    onPaneClick={onPaneClick}
   >

   {/* Toolbar */}
+  <button onClick={insertNodeOnEdge} disabled={!selectedEdge}>
+    Sett inn
+  </button>
-  <button onClick={saveFlow}>Lagre diagram</button>
+  <button onClick={saveFlow} disabled={!hasUnsavedChanges}>
+    {hasUnsavedChanges ? 'Lagre diagram' : 'Ingen endringer'}
+  </button>
```

**Forklaring:**
- "Sett inn"-knappen er kun aktiv når en edge er valgt (`selectedEdge` er satt)
- Lagre-knappen er deaktivert og viser "Ingen endringer" når `hasUnsavedChanges` er `false`
- `onEdgeClick` og `onPaneClick` er koblet til ReactFlow-komponentens event handlers

---

## Oppsummering av endringer

- **2 filer endret**, 144 linjer lagt til, 6 linjer slettet
- `insertNodeOnEdge` gir brukere mulighet til å sette inn noder mellom eksisterende noder i flowdiagrammer
- `hasUnsavedChanges` gir visuell tilbakemelding på lagringsstatus med deaktivert Lagre-knapp
- Edge-seleksjon implementert som grunnlag for insertNodeOnEdge-funksjonaliteten
