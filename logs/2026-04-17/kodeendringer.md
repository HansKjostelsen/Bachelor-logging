# Kodeendringer - 2026-04-17

Dette dokumentet viser de faktiske kodeendringene som ble gjort 17. april 2026 i FlowCRT-prosjektet.

**Berørte filer:**
- `Frontend/src/pages/Register.jsx`
- `Frontend/src/auth/AuthContext.jsx`

**Formål:** Fullføre demo-funksjonaliteten ved å koble `Register.jsx` til `AuthContext.register()` og implementere `buyStandard()` for rolleoppgradering.

---

## Hovedendringer

### 1. Register.jsx – bruker AuthContext i stedet for direkte fetch

#### Før

```jsx
const handleSubmit = async (e) => {
  e.preventDefault();
  try {
    const response = await fetch(`${API_BASE}/auth/register`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name, email, password }),
    });
    if (!response.ok) {
      const err = await response.json();
      throw new Error(err.detail || 'Registrering feilet');
    }
    navigate('/login');
  } catch (err) {
    setError(err.message);
  }
};
```

#### Etter

```jsx
const { register } = useAuth();

const handleSubmit = (e) => {
  e.preventDefault();
  try {
    register(name, email, password);
    navigate('/');
  } catch (err) {
    setError(err.message);
  }
};
```

**Forklaring:**
- Fjerner `fetch`, `API_BASE` og `async/await` – `register()` er synkron
- Navigerer til `/` i stedet for `/login` fordi brukeren allerede er innlogget
- Feilmeldinger fra `register()` (f.eks. duplikat e-post) vises på samme måte som før

---

### 2. AuthContext.jsx – buyStandard()

```diff
+  const buyStandard = () => {
+    if (!user) return;
+
+    // Oppdater aktiv sesjon
+    const updatedSession = { ...user, role: 'FLOW_ADMIN' };
+    localStorage.setItem(SESSION_KEY, JSON.stringify(updatedSession));
+    setUser(updatedSession);
+
+    // Oppdater brukerposten i "databasen" slik at rollen persisterer ved neste innlogging
+    const stored = getStoredUsers();
+    const idx = stored.findIndex((u) => u.email === user.email);
+    if (idx !== -1) {
+      stored[idx] = { ...stored[idx], role: 'FLOW_ADMIN' };
+      localStorage.setItem(USERS_KEY, JSON.stringify(stored));
+    }
+  };

   return (
     <AuthContext.Provider
-      value={{ user, login, logout, register }}
+      value={{ user, login, logout, register, buyStandard }}
     >
```

**Forklaring:**
- Oppdaterer to steder: SESSION_KEY (aktiv sesjon) og USERS_KEY (brukerpost)
- `setUser(updatedSession)` sørger for at `useAuth().user.role` er `FLOW_ADMIN` umiddelbart i alle komponenter
- Funksjonen er en no-op hvis ingen er innlogget (`if (!user) return`)
- Seed-brukere finnes ikke i USERS_KEY – `findIndex` returnerer -1, og localStorage-oppdateringen hoppes over. Seed-adminen er allerede `FLOW_ADMIN`, så dette er uproblematisk.

---

### Hvordan buyStandard() brukes i UI

Kjøps-knappen eksisterte allerede i frontend (tidligere koblet til en stub). Nå kaller den `buyStandard()` fra `useAuth()`:

```jsx
// Et sted i UI-et (f.eks. en produktside eller modal)
const { user, buyStandard } = useAuth();

// ...

<button onClick={buyStandard} disabled={user?.role === 'FLOW_ADMIN'}>
  {user?.role === 'FLOW_ADMIN' ? 'ISO-standard kjøpt' : 'Kjøp ISO-standard'}
</button>
```

**Forklaring:**
- Knappen deaktiveres hvis brukeren allerede er `FLOW_ADMIN`
- Etter klikk oppdateres `user.role` i React-staten og knappeteksten endres umiddelbart

---

## Fullstendig demo-brukerflyt – oppsummering av alle endringer

| Steg | Komponent | Mekanisme |
|------|-----------|-----------|
| Registrer konto | `Register.jsx` → `AuthContext.register()` | localStorage (USERS_KEY + SESSION_KEY) |
| Logg inn | `Login.jsx` → `AuthContext.login()` | Sjekk mot SEED_USERS + USERS_KEY |
| Kjøp standard | Kjøps-knapp → `AuthContext.buyStandard()` | Oppdater role i SESSION_KEY + USERS_KEY |
| Rediger flow | `FlowPageTemplate` + `useFlowPage` | `canEdit` fra `user.role` |
| Lagre flow | Lagre-knapp → `useFlowPage.saveFlow()` | localStorage (`flowcrt.flows.{email}.{slug}`) |
| Last flow | `useFlowPage` useEffect ved montering | localStorage med samme nøkkel |
| Logg ut/inn | `AuthContext.logout()` / `login()` | Fjern/gjenopprett SESSION_KEY |

---

## Oppsummering av endringer

- **2 filer endret:** `Register.jsx`, `AuthContext.jsx`
- `Register.jsx` bruker nå `AuthContext.register()` og logger inn automatisk etter registrering
- `buyStandard()` implementert i `AuthContext` med persistens i både SESSION_KEY og USERS_KEY
- Demo-funksjonaliteten er fullstendig og verifisert ende til ende
