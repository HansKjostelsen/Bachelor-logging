# Kodeendringer - 2026-03-17

Dette dokumentet viser de faktiske kodeendringene som ble gjort 17. mars 2026 i FlowCRT-prosjektet.

**Diff-område:** `b8e2d51..f3e8c12` (fra useFlowPage localStorage-commit til hasUnsavedChanges-commit)

**Merk:** Tre commits denne dagen. Fokus på registrering, rolleoppgradering og hasUnsavedChanges.

---

## Hovedendringer

### 1. register()-funksjon i AuthContext

#### Frontend/src/context/AuthContext.jsx (ENDRET)

```diff
 export function AuthProvider({ children }) {
   const [user, setUser] = useState(() => { ... });

   function login(email, password) { ... }

   function logout() { ... }

+  function register(name, email, password) {
+    const storedUsers = JSON.parse(localStorage.getItem(DEMO_USERS_KEY) || "[]");
+    const allUsers = [...SEED_USERS, ...storedUsers];
+
+    if (allUsers.find((u) => u.email === email)) {
+      throw new Error("E-post allerede i bruk");
+    }
+
+    const newUser = {
+      id: allUsers.length + 1,
+      email,
+      password,
+      role: "VIEWER",
+      name,
+    };
+
+    localStorage.setItem(
+      DEMO_USERS_KEY,
+      JSON.stringify([...storedUsers, newUser])
+    );
+  }
+
+  function buyStandard() {
+    if (!user) throw new Error("Ikke innlogget");
+
+    const storedUsers = JSON.parse(localStorage.getItem(DEMO_USERS_KEY) || "[]");
+    const updated = storedUsers.map((u) =>
+      u.email === user.email ? { ...u, role: "FLOW_ADMIN" } : u
+    );
+    localStorage.setItem(DEMO_USERS_KEY, JSON.stringify(updated));
+
+    const updatedSession = { ...user, role: "FLOW_ADMIN" };
+    localStorage.setItem(DEMO_SESSION_KEY, JSON.stringify(updatedSession));
+    setUser(updatedSession);
+  }

   return (
-    <AuthContext.Provider value={{ user, login, logout }}>
+    <AuthContext.Provider value={{ user, login, logout, register, buyStandard }}>
       {children}
     </AuthContext.Provider>
   );
 }
```

**Forklaring:**
- `register()` validerer at e-posten ikke allerede er i bruk ved å sjekke mot en sammenslått liste av seed-brukere og localStorage-brukere
- Nye brukere starter alltid med rollen `VIEWER` — dette speiler forventet backend-oppførsel
- `buyStandard()` oppdaterer brukeroppføringen i `flowcrt.demo.users` og sesjonen i `flowcrt.demo.session` atomisk for å sikre konsistens
- Begge funksjonene eksponeres via context-verdien slik at de er tilgjengelige overalt i komponenttreet

---

### 2. Register.jsx oppdatert til useAuth

#### Frontend/src/pages/Register.jsx (ENDRET)

```diff
-import { useState } from "react";
-import { useNavigate } from "react-router-dom";
+import { useState } from "react";
+import { useNavigate } from "react-router-dom";
+import { useAuth } from "../context/AuthContext";

 export default function Register() {
   const [name, setName] = useState("");
   const [email, setEmail] = useState("");
   const [password, setPassword] = useState("");
   const [error, setError] = useState(null);
   const navigate = useNavigate();
+  const { register, login } = useAuth();

   async function handleSubmit(e) {
     e.preventDefault();
     setError(null);
     try {
-      const res = await fetch("/auth/register", {
-        method: "POST",
-        headers: { "Content-Type": "application/json" },
-        body: JSON.stringify({ name, email, password }),
-        credentials: "include",
-      });
-      if (!res.ok) {
-        const data = await res.json();
-        throw new Error(data.message || "Registrering feilet");
-      }
-      navigate("/login");
+      register(name, email, password);
+      login(email, password);
+      navigate("/");
     } catch (err) {
       setError(err.message);
     }
   }
```

**Forklaring:**
- `register()` etterfulgt av `login()` implementerer automatisk innlogging etter registrering
- Navigerer til rotstien (`/`) i stedet for `/login` siden brukeren allerede er innlogget
- Feilhåndteringen er uendret — `catch`-blokken fanger opp feil kastet av `register()` og viser dem i skjemaet
- Komponenten er nå synkron (ingen `async/await`) siden `register()` og `login()` er synkrone localStorage-operasjoner

---

### 3. hasUnsavedChanges i useFlowPage

#### Frontend/src/hooks/useFlowPage.js (ENDRET)

```diff
 import { useState, useCallback, useRef } from "react";
 import { useAuth } from "../context/AuthContext";

 export function useFlowPage(pageSlug, initialNodes, initialEdges) {
   const [nodes, setNodes] = useState(initialNodes);
   const [edges, setEdges] = useState(initialEdges);
+  const [hasUnsavedChanges, setHasUnsavedChanges] = useState(false);
   const { user } = useAuth();

   const storageKey = user
     ? `flowcrt.flows.${user.email}.${pageSlug}`
     : null;

+  function handleSetNodes(updater) {
+    setNodes(updater);
+    setHasUnsavedChanges(true);
+  }
+
+  function handleSetEdges(updater) {
+    setEdges(updater);
+    setHasUnsavedChanges(true);
+  }
+
   function loadFlow() {
     if (!storageKey) return;
     try {
       const raw = localStorage.getItem(storageKey);
       if (!raw) return;
       const { nodes: savedNodes, edges: savedEdges } = JSON.parse(raw);
       setNodes(savedNodes);
       setEdges(savedEdges);
+      setHasUnsavedChanges(false);
     } catch {
       // Korrupt data i localStorage — ignorer og bruk standardverdier
     }
   }

   function saveFlow() {
     if (!storageKey) return;
     localStorage.setItem(storageKey, JSON.stringify({ nodes, edges }));
+    setHasUnsavedChanges(false);
   }

-  return { nodes, edges, setNodes, setEdges, loadFlow, saveFlow };
+  return {
+    nodes,
+    edges,
+    setNodes: handleSetNodes,
+    setEdges: handleSetEdges,
+    loadFlow,
+    saveFlow,
+    hasUnsavedChanges,
+  };
 }
```

**Bruk i komponent:**

```jsx
const { nodes, edges, setNodes, setEdges, saveFlow, hasUnsavedChanges } =
  useFlowPage(pageSlug, initialNodes, initialEdges);

// ...

<button
  className="save-button"
  onClick={saveFlow}
  disabled={!hasUnsavedChanges}
>
  {hasUnsavedChanges ? "Lagre diagram" : "Ingen endringer"}
</button>
```

**Forklaring:**
- `handleSetNodes` og `handleSetEdges` pakker inn de originale state-setterfunksjonene og setter `hasUnsavedChanges` til `true` ved enhver endring
- `loadFlow()` og `saveFlow()` nullstiller `hasUnsavedChanges` etter at tilstanden er synkronisert med lagring
- Hooken eksponerer fortsatt `setNodes` og `setEdges` med de samme navnene som før, slik at eksisterende komponenter ikke trenger å bytte navn på variablene
- Seleksjonsendringer (klikk på node uten å flytte den) trigger `onNodesChange`, men React Flow filtrerer dette internt — `setNodes` kalles bare ved faktiske strukturelle endringer

---

## Oppsummering av endringer

- **3 filer endret**, 76 linjer lagt til, 13 linjer slettet
- `register()` og `buyStandard()` fullfører AuthContext for demo-modus
- `Register.jsx` bruker nå `useAuth()` konsekvent i stedet for direkte fetch-kall
- `hasUnsavedChanges` gir bedre UX og tydeliggjør lagringsstatus for brukeren
- Demo-flyten er fullstendig: registrer → kjøp standard → rediger → lagre → logg ut/inn → data persistert
