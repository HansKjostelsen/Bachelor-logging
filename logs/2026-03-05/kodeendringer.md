# Kodeendringer - 2026-03-05

Dette dokumentet viser de faktiske kodeendringene som ble gjort 5. mars 2026 i FlowCRT-prosjektet.

**Diff-område:** `9a0e4f3..b8e2d51` (fra siste stabile commit til useFlowPage localStorage-commit)

**Merk:** To commits denne dagen. Fokus på demo-modus: localStorage-basert autentisering og flow-lagring.

---

## Hovedendringer

### 1. AuthContext omskrevet til localStorage-autentisering

#### Frontend/src/context/AuthContext.jsx (ENDRET)

```diff
-import { createContext, useContext, useState } from "react";
+import { createContext, useContext, useState, useEffect } from "react";

+const SEED_USERS = [
+  { id: 1, email: "admin@flowcrt.no", password: "admin123", role: "FLOW_ADMIN", name: "Admin Bruker" },
+  { id: 2, email: "bruker@flowcrt.no", password: "bruker123", role: "VIEWER", name: "Test Bruker" },
+];
+
+const DEMO_USERS_KEY = "flowcrt.demo.users";
+const DEMO_SESSION_KEY = "flowcrt.demo.session";

 const AuthContext = createContext(null);

 export function AuthProvider({ children }) {
-  const [user, setUser] = useState(null);
+  const [user, setUser] = useState(() => {
+    try {
+      const saved = localStorage.getItem(DEMO_SESSION_KEY);
+      return saved ? JSON.parse(saved) : null;
+    } catch {
+      return null;
+    }
+  });

-  async function login(email, password) {
-    const res = await fetch("/auth/login", {
-      method: "POST",
-      headers: { "Content-Type": "application/json" },
-      body: JSON.stringify({ email, password }),
-      credentials: "include",
-    });
-    if (!res.ok) throw new Error("Innlogging feilet");
-    const data = await res.json();
-    setUser(data.user);
+  function login(email, password) {
+    const storedUsers = JSON.parse(localStorage.getItem(DEMO_USERS_KEY) || "[]");
+    const allUsers = [...SEED_USERS, ...storedUsers];
+    const match = allUsers.find(
+      (u) => u.email === email && u.password === password
+    );
+    if (!match) throw new Error("Feil e-post eller passord");
+    const session = { id: match.id, email: match.email, role: match.role, name: match.name };
+    localStorage.setItem(DEMO_SESSION_KEY, JSON.stringify(session));
+    setUser(session);
   }

-  async function logout() {
-    await fetch("/auth/logout", { method: "POST", credentials: "include" });
-    setUser(null);
+  function logout() {
+    localStorage.removeItem(DEMO_SESSION_KEY);
+    setUser(null);
   }

-  return <AuthContext.Provider value={{ user, login, logout }}>{children}</AuthContext.Provider>;
+  return (
+    <AuthContext.Provider value={{ user, login, logout }}>
+      {children}
+    </AuthContext.Provider>
+  );
 }

 export function useAuth() {
   return useContext(AuthContext);
 }
```

**Forklaring:**
- `useState` initialiseres med en funksjon som leser fra localStorage, slik at innlogget tilstand overlever sideopplasting
- `SEED_USERS` hardkoder to testbrukere direkte i kildekoden — tilstrekkelig for demo, men må aldri brukes i produksjon
- `login()` slår opp mot en kombinasjon av seed-brukere og dynamisk registrerte brukere i `flowcrt.demo.users`
- `logout()` fjerner kun session-nøkkelen, ikke brukerdatabasen, slik at registrerte brukere overlever utlogging

---

### 2. useFlowPage oppdatert til localStorage-persistens

#### Frontend/src/hooks/useFlowPage.js (ENDRET)

```diff
-import { useState, useCallback } from "react";
+import { useState, useCallback, useRef } from "react";
+import { useAuth } from "../context/AuthContext";

 export function useFlowPage(pageSlug, initialNodes, initialEdges) {
   const [nodes, setNodes] = useState(initialNodes);
   const [edges, setEdges] = useState(initialEdges);
+  const { user } = useAuth();
+
+  const storageKey = user
+    ? `flowcrt.flows.${user.email}.${pageSlug}`
+    : null;

-  async function loadFlow() {
-    const res = await fetch(`/flows/${pageSlug}`, { credentials: "include" });
-    if (!res.ok) return;
-    const { nodes: savedNodes, edges: savedEdges } = await res.json();
-    setNodes(savedNodes);
-    setEdges(savedEdges);
-  }
+  function loadFlow() {
+    if (!storageKey) return;
+    try {
+      const raw = localStorage.getItem(storageKey);
+      if (!raw) return;
+      const { nodes: savedNodes, edges: savedEdges } = JSON.parse(raw);
+      setNodes(savedNodes);
+      setEdges(savedEdges);
+    } catch {
+      // Korrupt data i localStorage — ignorer og bruk standardverdier
+    }
+  }

-  async function saveFlow() {
-    await fetch(`/flows/${pageSlug}`, {
-      method: "POST",
-      headers: { "Content-Type": "application/json" },
-      credentials: "include",
-      body: JSON.stringify({ nodes, edges }),
-    });
+  function saveFlow() {
+    if (!storageKey) return;
+    localStorage.setItem(storageKey, JSON.stringify({ nodes, edges }));
   }

   return { nodes, edges, setNodes, setEdges, loadFlow, saveFlow };
 }
```

**Forklaring:**
- `storageKey` bygges dynamisk fra brukerens e-post og sidens slug, noe som sikrer at flows er isolert per bruker og per side
- `loadFlow()` er nå synkron siden localStorage-operasjoner ikke er asynkrone — ingen `async/await` nødvendig
- Try/catch rundt `JSON.parse` beskytter mot feil ved korrupt eller ugyldig data i localStorage
- Dersom brukeren ikke er innlogget (`user === null`) gjøres ingen ting, slik at uinnloggede brukere ikke kan lese eller skrive flows

---

## Oppsummering av endringer

- **2 filer endret**, 91 linjer lagt til, 56 linjer slettet
- `AuthContext.jsx` håndterer nå innlogging, sesjonspersistens og utlogging utelukkende via localStorage
- `useFlowPage.js` lagrer og henter flows per bruker og per side via localStorage
- Applikasjonen kan nå kjøres i sin helhet uten backend og demonstrere full brukerflyt
- Neste steg er å legge til registreringsfunksjonalitet og `buyStandard()` for rolleoppgradering
