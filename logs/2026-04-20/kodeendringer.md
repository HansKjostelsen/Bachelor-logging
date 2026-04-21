# Kodeendringer - 2026-04-20

Dette dokumentet viser de faktiske kodeendringene som ble gjort 20. april 2026 i FlowCRT-prosjektet.

**Diff-område:** `b45bed77..8682af77` (fra list_user_access-fix til JWT-migrering)

**Merk:** Mange commits denne dagen fra alle gruppemedlemmer. Fokus på tre hovedområder: backend-autentisering (session→JWT), frontend flow-redigering (insertNodeOnEdge) og adminpanel.

---

## Hovedendringer

### 1. list_user_access() implementert

#### API/src/repositories/users.py (ENDRET)

```diff
-def list_user_access(user_id: int):
-    # Python NEEDS at least one indented line here.
-    # If you haven't written the logic yet, use 'pass'
-    pass
+def list_user_access(user_id: int):
+    with engine.connect() as conn:
+        result = conn.execute(
+            text("""
+                SELECT d.pk_domain_id AS domain_id, d.company_name, auth_lvl.authorization_name AS role
+                FROM user_access_to_domains u_access
+                JOIN domains d ON u_access.fk_domain_id = d.pk_domain_id
+                JOIN authorization_levels auth_lvl ON u_access.fk_authorization_level_id = auth_lvl.pk_authorization_level_id
+                WHERE u_access.fk_user_id = :user_id
+            """),
+            {"user_id": user_id},
+        )
+        return [dict(row._mapping) for row in result]
```

**Forklaring:**
- Funksjonen var tidligere tom (`pass`) — nå returnerer den en liste over domener brukeren har tilgang til
- SQL-spørringen joiner tre tabeller: `user_access_to_domains`, `domains` og `authorization_levels`
- Resultatet inneholder `domain_id`, `company_name` og `role` per domene, som brukes av `/me`-endepunktet og JWT-payload

---

### 2. insertNodeOnEdge og hasUnsavedChanges

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
+    const newX = (sourceNode.position.x + targetNode.position.x) / 2;
+    const newY = (sourceNode.position.y + targetNode.position.y) / 2;
+
+    const shiftX = (targetNode.position.x - sourceNode.position.x) / 2;
+    const shiftY = (targetNode.position.y - sourceNode.position.y) / 2;
+
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
+    // Erstatt gammel edge med to nye
+    const edgeToNew = { ... source→newNode };
+    const edgeFromNew = { ... newNode→target };
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

**Forklaring:**
- `insertNodeOnEdge` beregner midtpunktet mellom source og target for å plassere den nye noden, og forskyver target-noden med halve den opprinnelige avstanden slik at det blir jevnt gap
- `oppositeHandle`-mappingen sikrer at den nye nodens innkommende handle matcher den utgående handle-retningen fra source
- `hasUnsavedChanges` filtrerer bort `select`- og `dimensions`-endringer i `onNodesChange` for å unngå falske positive — kun faktiske posisjons- og strukturendringer trigger ulagret-tilstand
- `onEdgeClick` og `onPaneClick` muliggjør edge-seleksjon og deseleksjon, og er gjensidig eksklusiv med node-seleksjon

---

### 3. Adminpanel med React Admin

#### Frontend/src/admin/AdminPanel.jsx (NY FIL)

```jsx
import { Admin, Resource, List, Datagrid, TextField, EmailField,
         Create, Edit, SimpleForm, TextInput, SelectInput, NumberInput } from 'react-admin';
import fakeRestProvider from 'ra-data-fakerest';
import { BrowserRouter } from 'react-router-dom';
import data from './data';

const UserList = () => (
    <List>
        <Datagrid rowClick="edit">
            <TextField source="id" />
            <TextField source="name" />
            <EmailField source="email" />
            <TextField source="role" />
        </Datagrid>
    </List>
);

const roleChoices = [
    { id: 'EMPLOYEE', name: 'Ansatt' },
    { id: 'FLOW_ADMIN', name: 'Flow Eier' },
    { id: 'SYSTEM_ADMIN', name: 'System Admin' },
    { id: 'VISITOR', name: 'Gjest' },
    { id: 'CERT_AGENT', name: 'Sertifiserings agent' },
];

const UserCreate = () => ( <Create><SimpleForm>...</SimpleForm></Create> );
const UserEdit = () => ( <Edit><SimpleForm>...</SimpleForm></Edit> );

const ProductList = () => ( <List><Datagrid>...</Datagrid></List> );
const ProductCreate = () => ( <Create><SimpleForm>...</SimpleForm></Create> );
const ProductEdit = () => ( <Edit><SimpleForm>...</SimpleForm></Edit> );

const AdminPanel = () => (
    <BrowserRouter basename="/admin">
        <Admin dataProvider={fakeRestProvider(data)}>
            <Resource name="users" list={UserList} create={UserCreate} edit={UserEdit} />
            <Resource name="products" list={ProductList} create={ProductCreate} edit={ProductEdit} />
        </Admin>
    </BrowserRouter>
);

export default AdminPanel;
```

#### Frontend/src/admin/data.js (NY FIL)

```js
const data = {
  users: [
    { id: 1, name: "Ola Nordmann", email: "ola@test.no", role: "EMPLOYEE" },
    { id: 2, name: "Kari Hansen", email: "kari@test.no", role: "FLOW_ADMIN" },
    { id: 3, name: "Admin Bruker", email: "admin@test.no", role: "SYSTEM_ADMIN" },
  ],
  products: [
    { id: 1, name: "ISO 9001", description: "Kvalitetsstyring", price: 5000 },
    { id: 2, name: "ISO 14001", description: "Miljøstyring", price: 4500 },
    { id: 3, name: "ISO 27001", description: "Informasjonssikkerhet", price: 6000 },
  ],
};
export default data;
```

#### Frontend/src/App.jsx (ENDRET)

```diff
+import AdminPanel from "./admin/AdminPanel";

 function App() {
+  if (window.location.pathname.startsWith("/admin")) {
+    return <AdminPanel />;
+  }
+
   const protectFlowRoute = (element, requiredStandard) => (
```

**Forklaring:**
- Adminpanelet bruker `react-admin` med `fakeRestProvider` som datakilde — all CRUD skjer mot hardkodet testdata i `data.js`
- Eget `BrowserRouter` med `basename="/admin"` isolerer admin-rutene fra hovedapplikasjonen
- App.jsx sjekker URL-stien og rendrer `AdminPanel` separat dersom man er under `/admin`-stien — dette unngår konflikter mellom de to routerne
- Rollene i admin-dropdown samsvarer med FlowCRT sine 5 roller definert i `roles.js`

---

### 4. Migrering fra session til JWT

#### API/src/repositories/sessions.py (SLETTET)

```diff
-def create_session(session_id, user_id, expires_at):
-    # INSERT INTO user_sessions ...
-
-def get_session_with_user(session_id):
-    # SELECT ... FROM user_sessions JOIN users ...
-
-def delete_session(session_id):
-    # DELETE FROM user_sessions WHERE pk_user_session_id = :sid
```

#### API/.env (ENDRET)

```diff
-SESSION_EXPIRE_MINUTES=60
+COOKIE_SECURE=false
+JWT_SECRET=ae17bb9e...  # 256-bit hex-nøkkel
+JWT_ALGORITHM=HS256
+JWT_EXPIRE_MINUTES=60
```

#### API/requirements.txt (ENDRET)

```diff
 uvicorn==0.40.0
+python-jose[cryptography]==3.3.0
```

#### API/src/app_config.py (ENDRET)

```diff
-SESSION_EXPIRE_MINUTES = int(os.getenv("SESSION_EXPIRE_MINUTES", 60))
+JWT_SECRET = os.getenv("JWT_SECRET")
+JWT_ALGORITHM = os.getenv("JWT_ALGORITHM", "HS256")
+JWT_EXPIRE_MINUTES = int(os.getenv("JWT_EXPIRE_MINUTES", 60))
+
+if not JWT_SECRET:
+    raise RuntimeError("JWT_SECRET mangler i .env")
```

#### API/src/deps/auth.py (ENDRET)

```diff
-from ..repositories.sessions import delete_session, get_session_with_user
-from ..repositories.users import list_user_access
+from jose import JWTError, jwt
+from .. import app_config

 def get_current_user(request: Request):
-    raw_session_id = request.cookies.get("session_id")
-    if not raw_session_id:
+    token = request.cookies.get("access_token")
+    if not token:
         raise HTTPException(status_code=401, detail="Not authenticated")

     try:
-        session_id = int(raw_session_id)
-    except ValueError:
-        raise HTTPException(...)
-
-    session = get_session_with_user(session_id)
-    if not session:
-        raise HTTPException(...)
-
-    if session["expires_at"] <= datetime.utcnow():
-        delete_session(session_id)
-        raise HTTPException(...)
-
-    access = list_user_access(session["user_id"])
+        payload = jwt.decode(token, app_config.JWT_SECRET, algorithms=[app_config.JWT_ALGORITHM])
+    except JWTError:
+        raise HTTPException(status_code=401, detail="Invalid or expired token")

     return {
-        "id": session["user_id"],
-        "email": session["email"],
-        "access": access,
+        "id": payload["sub"],
+        "email": payload["email"],
+        "access": payload["access"],
     }
```

**Forklaring:**
- `sessions.py` er fullstendig fjernet — ingen databasebaserte sesjoner lenger
- `get_current_user()` leser nå `access_token`-cookien og dekoder JWT-tokenet kryptografisk med `jose.jwt.decode()`
- Brukerdata (id, email, access/roller) er innebygd i JWT-payload i stedet for å slås opp i databasen
- `app_config.py` krever at `JWT_SECRET` er satt i `.env` — uten dette kaster applikasjonen `RuntimeError` ved oppstart
- `python-jose[cryptography]` brukes for JWT-operasjoner, med HS256 som signeringsalgoritme

---

## Oppsummering av endringer

- **~12 filer endret**, ~1700 linjer lagt til (mesteparten package-lock), ~170 linjer slettet
- Autentisering migrert fra session-basert (database) til JWT (stateless) — `sessions.py` slettet
- `list_user_access()` implementert med SQL-spørring for rollebasert tilgangskontroll
- `insertNodeOnEdge` gir brukere mulighet til å sette inn noder mellom eksisterende noder i flowdiagrammer
- `hasUnsavedChanges` gir visuell tilbakemelding på lagringsstatus
- Adminpanel med React Admin for bruker- og produktadministrasjon (fakeRestProvider)
- Gruppearbeid: Hans (frontend), Sondre (backend/auth), Henrik (adminpanel), Felix (database)
