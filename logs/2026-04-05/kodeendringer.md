# Kodeendringer - 2026-04-05

Dette dokumentet viser de faktiske kodeendringene som ble gjort 5. april 2026 i FlowCRT-prosjektet.

**Berørt fil:** `Frontend/src/layout/useFlowPage.js`

**Formål:** Erstatte API-kall med localStorage-lagring, innføre per-bruker nøkler og legge til `hasUnsavedChanges`-tilstand.

---

## Hovedendringer

### useFlowPage.js – localStorage erstatter API

#### Nøkkelgenerering og import

```diff
+import { useAuth } from '../auth/AuthContext';

-const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || 'http://localhost:8000';
+const storageKey = (email, slug) => `flowcrt.flows.${email}.${slug}`;
```

**Forklaring:**
- `API_BASE_URL`-konstanten er ikke lenger nødvendig og fjernes
- `storageKey` er en ren funksjon som genererer en unik og lesbar localStorage-nøkkel per bruker og per side

---

#### Innlasting av lagret flow

```diff
-export function useFlowPage(pageTitle, templateNodes = null, templateEdges = null, canEdit = true, pageSlug = null) {
+export function useFlowPage(pageTitle, templateNodes = null, templateEdges = null, canEdit = true, pageSlug = null) {
+  const { user } = useAuth();

+  const [hasUnsavedChanges, setHasUnsavedChanges] = useState(false);

   // Load saved flow on mount
   useEffect(() => {
-    if (!pageSlug) return;
-    fetch(`${API_BASE_URL}/flows/${pageSlug}`, { credentials: 'include' })
-      .then((r) => (r.ok ? r.json() : null))
-      .then((saved) => {
-        if (!saved?.nodes) return;
-        setNodes(applyCanEdit(saved.nodes));
-        if (saved.edges) setEdges(saved.edges);
-      })
-      .catch(() => {});
+    if (!pageSlug || !user?.email) return;
+    try {
+      const raw = localStorage.getItem(storageKey(user.email, pageSlug));
+      if (!raw) return;
+      const saved = JSON.parse(raw);
+      if (saved?.nodes) setNodes(applyCanEdit(saved.nodes));
+      if (saved?.edges) setEdges(saved.edges);
+    } catch {}
-  }, [pageSlug]);
+  }, [pageSlug, user?.email]);
```

**Forklaring:**
- `user?.email` brukes som del av nøkkelen – hooks kjøres på nytt når innlogget bruker endres
- Defensiv try/catch sikrer at korrupt JSON i localStorage ikke krasjer siden
- `applyCanEdit` kalles fortsatt på innlastede noder slik at rollen alltid respekteres

---

#### hasUnsavedChanges i onNodesChange og onEdgesChange

```diff
   const onNodesChange = useCallback(
-    (changes) => setNodes((nds) => applyNodeChanges(changes, nds)),
+    (changes) => {
+      const meaningful = changes.some((c) => c.type !== 'select');
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
```

**Forklaring:**
- For noder filtreres `select`-endringer ut – et enkelt klikk skal ikke aktivere lagre-knappen
- Alle kantendringer regnes som meningsfulle (kanter kan ikke velges uten å påvirke data)

---

#### saveFlow – skriv til localStorage

```diff
   const saveFlow = useCallback(async () => {
-    if (!pageSlug) return;
-    await fetch(`${API_BASE_URL}/flows/${pageSlug}`, {
-      method: 'POST',
-      headers: { 'Content-Type': 'application/json' },
-      credentials: 'include',
-      body: JSON.stringify({ nodes, edges }),
-    });
+    if (!pageSlug || !user?.email) return;
+    localStorage.setItem(
+      storageKey(user.email, pageSlug),
+      JSON.stringify({ nodes, edges })
+    );
+    setHasUnsavedChanges(false);
   }, [pageSlug, nodes, edges, user?.email]);

   return {
     // ... eksisterende ...
+    hasUnsavedChanges,
+    saveFlow,
   };
```

**Forklaring:**
- `localStorage.setItem` er synkron – ingen `async`/`await` nødvendig
- `hasUnsavedChanges` settes til `false` etter lagring slik at knappen igjen deaktiveres
- `user?.email` er med i dependency-arrayen til `useCallback` for at `saveFlow` alltid bruker riktig nøkkel

---

#### hasUnsavedChanges brukt i FlowPageTemplate

```diff
   const {
     nodes, edges, onNodesChange, onEdgesChange, onConnect,
     onNodeClick, onNodeDoubleClick, addNodeInDirection, saveFlow,
-    goBack, setRfInstance,
+    goBack, setRfInstance, hasUnsavedChanges,
   } = useFlowPage(pageTitle, templateNodes, templateEdges, canEdit, pageSlug);

   // ...

       {canEdit && pageSlug && (
         <button
-          style={{ ...backButtonStyle, marginLeft: '10px', background: '#1a5c1a' }}
+          style={{
+            ...backButtonStyle,
+            marginLeft: '10px',
+            background: hasUnsavedChanges ? '#1a5c1a' : '#555',
+            cursor: hasUnsavedChanges ? 'pointer' : 'not-allowed',
+            opacity: hasUnsavedChanges ? 1 : 0.6,
+          }}
           onClick={saveFlow}
+          disabled={!hasUnsavedChanges}
         >
           Lagre
         </button>
       )}
```

**Forklaring:**
- Knappen er deaktivert og visuelt nedtonet når ingen endringer er gjort
- Fargen endres fra grå (`#555`) til grønn (`#1a5c1a`) når det finnes ulagrede endringer – et tydelig visuelt signal
- `disabled`-attributtet hindrer klikk og `cursor: not-allowed` gir riktig musepeker

---

## Oppsummering av endringer

- **1 fil endret:** `Frontend/src/layout/useFlowPage.js`
- **1 fil endret:** `Frontend/src/pages/ISO-sider/9001-Undersider/Dynamiske/FlowPageTemplate.jsx`
- API-kall til `GET/POST /flows/{pageSlug}` erstattet med localStorage
- Per-bruker nøkkel: `flowcrt.flows.{email}.{slug}`
- `hasUnsavedChanges`-state innført for å styre lagre-knappens tilstand
- `select`-endringer på noder utløser ikke lenger `hasUnsavedChanges = true`
