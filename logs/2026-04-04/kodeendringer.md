# Kodeendringer - 2026-04-04

Dette dokumentet viser de faktiske kodeendringene som ble gjort 4. april 2026 i FlowCRT-prosjektet.

**Berørt fil:** `Frontend/src/auth/AuthContext.jsx`

**Formål:** Fjerne alle API-kall fra autentiseringslaget og erstatte dem med localStorage-basert logikk, slik at frontend kan demonstreres uten backend.

---

## Hovedendringer

### AuthContext.jsx – fra API til localStorage

#### Seed-brukere og initial state

```diff
+const SEED_USERS = [
+  { id: 1, name: 'Demo Bruker',  email: 'bruker@demo.no',  password: 'demo123',  role: 'USER' },
+  { id: 2, name: 'Demo Admin',   email: 'admin@demo.no',   password: 'admin123', role: 'FLOW_ADMIN' },
+];
+
+const USERS_KEY = 'flowcrt.demo.users';
+const SESSION_KEY = 'flowcrt.demo.session';
+
+function getStoredUsers() {
+  try { return JSON.parse(localStorage.getItem(USERS_KEY) || '[]'); }
+  catch { return []; }
+}
```

**Forklaring:**
- `SEED_USERS` er alltid tilgjengelig, uavhengig av localStorage
- `USERS_KEY` og `SESSION_KEY` er konstante nøkler for å unngå skrivefeil
- `getStoredUsers()` er defensiv – en korrupt JSON-verdi i localStorage krasjer ikke appen

---

#### login() – lokal autentisering

```diff
-  const login = async (email, password) => {
-    const response = await fetch(`${API_BASE}/auth/login`, {
-      method: 'POST',
-      headers: { 'Content-Type': 'application/json' },
-      credentials: 'include',
-      body: JSON.stringify({ email, password }),
-    });
-    if (!response.ok) throw new Error('Feil e-post eller passord');
-    const data = await response.json();
-    setUser(data.user);
-  };
+  const login = (email, password) => {
+    const allUsers = [...SEED_USERS, ...getStoredUsers()];
+    const found = allUsers.find(
+      (u) => u.email === email && u.password === password
+    );
+    if (!found) throw new Error('Feil e-post eller passord');
+    const session = { id: found.id, name: found.name, email: found.email, role: found.role };
+    localStorage.setItem(SESSION_KEY, JSON.stringify(session));
+    setUser(session);
+  };
```

**Forklaring:**
- Søker gjennom seed-brukere og localStorage-brukere samlet
- Kaster feil med samme melding som backend-versjonen, slik at `Login.jsx` ikke trenger endringer
- Lagrer sessionen i localStorage så brukeren forblir innlogget ved sideoppdatering

---

#### register() – ny brukerfunksjon

```diff
+  const register = (name, email, password) => {
+    const allUsers = [...SEED_USERS, ...getStoredUsers()];
+    if (allUsers.find((u) => u.email === email)) {
+      throw new Error('E-postadressen er allerede registrert');
+    }
+    const newUser = {
+      id: Date.now(),
+      name,
+      email,
+      password,
+      role: 'USER',
+    };
+    const stored = getStoredUsers();
+    stored.push(newUser);
+    localStorage.setItem(USERS_KEY, JSON.stringify(stored));
+    const session = { id: newUser.id, name, email, role: 'USER' };
+    localStorage.setItem(SESSION_KEY, JSON.stringify(session));
+    setUser(session);
+  };
```

**Forklaring:**
- Nye brukere får alltid rollen `USER` – rolleoppgradering skjer separat via `buyStandard()`
- `id: Date.now()` er tilstrekkelig for demo-formål (unik innenfor én nettleser)
- Brukeren logges inn automatisk etter registrering uten ekstra kall

---

#### logout() og sessjonsgjenoppretting

```diff
-  const logout = async () => {
-    await fetch(`${API_BASE}/auth/logout`, { method: 'POST', credentials: 'include' });
-    setUser(null);
-  };
+  const logout = () => {
+    localStorage.removeItem(SESSION_KEY);
+    setUser(null);
+  };

+  // Gjenopprett sesjon ved sideinnlasting
+  useEffect(() => {
+    try {
+      const saved = localStorage.getItem(SESSION_KEY);
+      if (saved) setUser(JSON.parse(saved));
+    } catch {
+      localStorage.removeItem(SESSION_KEY);
+    }
+  }, []);
```

**Forklaring:**
- `logout()` trenger ikke lenger å være async siden det kun er lokale operasjoner
- `useEffect` ved montering erstatter det tidligere `/me`-kallet som hentet innlogget bruker fra backend
- Defensiv try/catch i `useEffect` sørger for at en korrupt sesjonsnøkkel ikke hindrer lasting

---

## Oppsummering av endringer

- **1 fil endret:** `Frontend/src/auth/AuthContext.jsx`
- Alle API-kall fjernet fra autentiseringslaget
- Innført `SEED_USERS` for alltid tilgjengelige testkontoer
- Ny `register()`-funksjon for å opprette brukere lokalt
- Sessjonspersistens via `localStorage` erstatter HTTPOnly-cookie-sesjonen fra backend
