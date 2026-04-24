# Kodeendringer - 2026-04-22

Dette dokumentet viser de faktiske kodeendringene som ble gjort 22. april 2026 i FlowCRT-prosjektet.

**Diff-område:** 7f800e79..5a611be4 (3 egne commits)

---

## 1. Ny FreeCanvas-side

### Frontend/src/pages/FreeCanvas.jsx (NY FIL)

```jsx
export default function FreeCanvas() {
  const { user } = useAuth();
  const navigate = useNavigate();
  const canEdit = user?.role === 'FLOW_ADMIN' || user?.role === 'SYSTEM_ADMIN';

  const {
    nodes, edges, selectedNode, hasUnsavedChanges,
    onNodesChange, onEdgesChange, onNodeClick,
    addNodeInDirection, updateNodeLabel, updateNodeDescription,
    saveFlow, setRfInstance,
  } = useFlowPage('Min Canvas', null, null, canEdit, 'free-canvas');

  const [editMenuOpen, setEditMenuOpen] = useState(false);
  const editMenuRef = useRef(null);

  // Lukk dropdown ved klikk utenfor
  useEffect(() => {
    if (!editMenuOpen) return;
    const handleClickOutside = (e) => {
      if (editMenuRef.current && !editMenuRef.current.contains(e.target)) {
        setEditMenuOpen(false);
      }
    };
    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, [editMenuOpen]);

  const handleEditOption = useCallback((action) => {
    setEditMenuOpen(false);
    if (!selectedNode) return;
    const nodeId = selectedNode.id;
    if (action === 'label') {
      const newLabel = window.prompt('Endre nodenavn:', selectedNode.data.label);
      if (newLabel && newLabel.trim()) updateNodeLabel(nodeId, newLabel.trim());
    } else if (action === 'description') {
      const newDesc = window.prompt('Endre kort info:', selectedNode.data.description || '');
      if (newDesc !== null) updateNodeDescription(nodeId, newDesc.trim());
    }
  }, [selectedNode, updateNodeLabel, updateNodeDescription]);
  // ...
}
```

**Forklaring:**
- `FreeCanvas` er en selvstendig side registrert på ruten `/canvas`. Den bruker `useFlowPage` med slug `free-canvas`, slik at flyten lagres per bruker i API-et
- Støtter kun `label`- og `description`-redigering — ikke `slug`/underprosess, siden FreeCanvas ikke er koblet til ISO-undersider
- Designet er identisk med `FlowPageTemplate` (samme farger, toolbar, wrapper-stil)

---

## 2. Redigeringsmeny i FlowPageTemplate

### Frontend/src/pages/ISO-sider/9001-Undersider/Dynamiske/FlowPageTemplate.jsx (ENDRET)

**Fjernet "Sett inn"-knapp, erstattet med "Rediger ▾"-dropdown:**

```diff
-  <button
-    style={directionBtnStyle(!selectedEdge)}
-    onClick={insertNodeOnEdge}
-    disabled={!selectedEdge}
-  >
-    ✦ Sett inn
-  </button>
+  <div ref={editMenuRef} style={{ position: 'relative', display: 'inline-block' }}>
+    <button
+      style={directionBtnStyle(!selectedNode)}
+      onClick={() => setEditMenuOpen((prev) => !prev)}
+      disabled={!selectedNode}
+    >
+      Rediger ▾
+    </button>
+    {editMenuOpen && selectedNode && (
+      <div style={{
+        position: 'absolute', top: '100%', left: 0, marginTop: '4px',
+        background: '#013220', border: '2px solid #014d01',
+        borderRadius: '8px', zIndex: 100, minWidth: '200px',
+      }}>
+        {[
+          { key: 'label', text: 'Endre navn' },
+          { key: 'description', text: 'Endre kort info' },
+          { key: 'slug', text: 'Sett som underprosess' },
+        ].map((item) => (
+          <button key={item.key} onClick={() => handleEditOption(item.key)} ...>
+            {item.text}
+          </button>
+        ))}
+      </div>
+    )}
+  </div>
```

**Ny handleEditOption-funksjon:**

```jsx
const handleEditOption = useCallback((action) => {
  setEditMenuOpen(false);
  if (!selectedNode) return;
  const nodeId = selectedNode.id;

  if (action === 'label') {
    const newLabel = window.prompt('Endre nodenavn:', selectedNode.data.label);
    if (newLabel && newLabel.trim()) updateNodeLabel(nodeId, newLabel.trim());

  } else if (action === 'description') {
    const newDesc = window.prompt('Endre kort info (vises ved hover):', selectedNode.data.description || '');
    if (newDesc !== null) updateNodeDescription(nodeId, newDesc.trim());

  } else if (action === 'slug') {
    const defaultSlug = selectedNode.data.slug
      || selectedNode.data.label.toLowerCase().replace(/\s+/g, '-').replace(/[^a-z0-9-]/g, '');
    const newSlug = window.prompt('Slug for underprosess:\nTom verdi fjerner lenken:', defaultSlug);
    if (newSlug !== null) updateNodeSlug(nodeId, newSlug.trim());
  }
}, [selectedNode, updateNodeLabel, updateNodeDescription, updateNodeSlug]);
```

**Dynamisk tilbakeknapp:**

```diff
- Tilbake til ISO 9001
+ Tilbake til {backLabel}
```

```jsx
const backLabel = location.state?.fromTitle || 'ISO 9001';
```

**Forklaring:**
- `handleEditOption` bruker `window.prompt` for enkel redigering uten modal-kompleksitet
- Dropdown bruker `position: absolute` med z-index 100 for å legge seg over flyt-canvaset
- `fromTitle` i `location.state` gjør tilbakeknappen dynamisk — brukes når man navigerer inn fra en underside

---

## 3. AuthContext — kjøpslogikk og purchasedStandards

### Frontend/src/auth/AuthContext.jsx (ENDRET)

**mapMeToUser henter purchased_products fra backend:**

```diff
- const purchasedStandards = Array.isArray(me?.purchased_products)
-   ? me.purchased_products
-   : []
-
  return {
    email: me?.email || fallbackEmail,
    role: primaryRole,
    companyIds: access.map((entry) => entry.domain_id).filter((id) => Number.isInteger(id)),
    access,
-   purchasedStandards,
+   purchasedStandards: Array.isArray(me?.purchased_products) ? me.purchased_products : [],
  }
```

**buyStandard — ny flyt med domainId:**

```diff
- const buyStandard = async (standardCode, companyName) => {
+ const buyStandard = async ({ productCode, companyName, domainId }) => {

-   const buyRes = await fetch(`${API_BASE_URL}/auth/buy-flow`, { ... })
+   if (!domainId) {
+     const buyFlowRes = await fetch(`${API_BASE_URL}/auth/buy-flow`, {
+       body: JSON.stringify({ company_name: companyName }),
+     })
+     if (!buyFlowRes.ok) return { ok: false }
+   }
+
+   const buyProductBody = { product_code: productCode }
+   if (domainId) buyProductBody.domain_id = domainId
+
+   const buyProductRes = await fetch(`${API_BASE_URL}/auth/buy-product`, {
+     body: JSON.stringify(buyProductBody),
+   })
```

**Forklaring:**
- Betingelsen er nå `!domainId` i stedet for `isVisitor` — mer generell, lar FLOW_ADMIN også opprette nye bedrifter
- `purchasedStandards` hentes alltid friskt fra `/me` etter kjøp — ingen lokal akkumulering
- `buyProductBody` bygges dynamisk: `domain_id` inkluderes kun når en eksisterende bedrift er valgt

---

## 4. PurchaseStandards — produkt-routing og buyOption

### Frontend/src/pages/PurchaseStandards.jsx (ENDRET)

**STANDARD_CATALOG utvidet:**

```diff
  { code: 'ISO9001', title: 'ISO 9001', description: '...',
+   buyOption: 'template', route: '/iso9001' },
  { code: 'ISO14001', title: 'ISO 14001', description: '...',
+   buyOption: 'template', route: '/iso14001' },
  { code: 'ISO27001', title: 'ISO 27001', description: '...',
+   buyOption: 'template', route: '/iso27001' },
  { code: 'CANVAS', title: 'Canvas', description: '...', subscription: true,
+   buyOption: 'empty', route: '/canvas' },
```

**handleBuy sender buyOption:**

```diff
  const handleBuy = async (standard) => {
-   const payload = { productCode: standard.code }
+   const payload = {
+     productCode: standard.code,
+     buyOption: standard.buyOption,
+   }
```

**Navigering til riktig rute:**

```diff
- onClick={() => navigate('/iso9001')}
+ onClick={() => navigate(purchasedProduct?.route || '/iso9001')}
```

**Forklaring:**
- `buyOption: 'template'` indikerer at backend skal opprette en side med malnoder. `buyOption: 'empty'` gir en tom canvas
- `route` gjør at bekreftelsessiden sender brukeren til riktig URL etter kjøp, uavhengig av produkt

---

## 5. JSDoc-kommentering (8d9c43bb)

### Frontend/src/auth/useFlowPage.js (ENDRET)

```diff
 /**
- * Custom hook for ISO 9001 sub-pages
- * Handles all the React Flow logic: nodes, edges, callbacks, and fitView
+ * Custom hook som håndterer all React Flow-logikk for ISO 9001-undersider.
+ * Tar hånd om nodes, edges, redigering, lagring til API og navigasjon.
  *
- * @param {string} pageTitle - The title/label for the initial node
- * @returns {object} - All the state, callbacks, and refs needed for a flow page
+ * @param {string} pageTitle - Tittelen som vises på startnoden (f.eks. "Anskaffelse")
+ * @param {Array|null} templateNodes - Forhåndsdefinerte noder, eller null for standardoppsett
+ * @param {Array|null} templateEdges - Forhåndsdefinerte kanter, eller null for tom liste
+ * @param {boolean} canEdit - Om brukeren har rettighet til å redigere flyten
+ * @param {string|null} pageSlug - URL-slug for denne siden, brukes til API-lagring
+ * @returns {object} State, callbacks og refs som FlowPageTemplate trenger
  */
```

```diff
+ // Henter lagret flyt fra API når siden lastes — bruker domainId for multi-tenant isolasjon
  useEffect(() => { ... }, [pageSlug, domainId]);

+ // Sender alle noder og kanter som JSON til API for lagring
  const saveFlow = useCallback(async () => { ... });

+ // Tilpasser visningen til vindusstørrelsen når brukeren endrer størrelse
  useEffect(() => { ... }, [rfInstance]);

+ // Tilpasser visningen automatisk når antall noder endres — timeout gir React Flow tid til å beregne dimensjoner
  useEffect(() => { ... }, [nodes.length, rfInstance]);
```

**Forklaring:**
- Kommentarene er skrevet på norsk for konsistens med resten av prosjektdokumentasjonen
- JSDoc-parameterlisten beskriver nå alle fem parametre, noe som gir bedre IDE-støtte og leserforståelse

---

## Oppsummering av endringer

- **14 filer berørt** (inkludert 1 ny fil: `FreeCanvas.jsx`)
- Ca. **1050 linjer lagt til**, ca. **300 linjer slettet**
- `FreeCanvas.jsx` er en ny, frittstående side for tom arbeidsflate med slug-basert persistens
- Redigeringsmenyen i `FlowPageTemplate` er nå dropdown-basert og aktiveres ved valgt node
- `purchasedStandards` er nå alltid synkronisert mot backend via `/me`
- `buyOption` og `route` er lagt til i `STANDARD_CATALOG` for korrekt backend-ruting og navigering
- All frontend-kode er kommentert etter JSDoc-standard
