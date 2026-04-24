# Kodeendringer - 2026-04-23

Dette dokumentet viser de faktiske kodeendringene som ble gjort 23. april 2026 i FlowCRT-prosjektet.

**Diff-område:** b40ee587 (én commit)

---

## 1. AuthContext — session-basert auth og sessionStorage-buffer

### Frontend/src/auth/AuthContext.jsx (ENDRET)

**Ny sessionStorage-hjelpefunksjoner:**

```js
const PRODUCTS_KEY = 'flowcrt.products'

function readStoredProducts(email) {
  try {
    const raw = sessionStorage.getItem(PRODUCTS_KEY)
    if (!raw) return []
    const parsed = JSON.parse(raw)
    return parsed?.email === email && Array.isArray(parsed.products)
      ? parsed.products
      : []
  } catch { return [] }
}

function writeStoredProducts(email, products) {
  if (!email) return
  sessionStorage.setItem(PRODUCTS_KEY, JSON.stringify({ email, products }))
}

function clearStoredProducts() {
  sessionStorage.removeItem(PRODUCTS_KEY)
}
```

**mapMeToUser — merger backend og sessionStorage:**

```diff
- function mapMeToUser(me, fallbackEmail = '') {
+ function mapMeToUser(me, knownProducts = []) {
    ...
-   const purchasedStandards = Array.isArray(me?.purchased_products)
-     ? me.purchased_products
-     : []
+   const fromBackend = Array.isArray(me?.purchased_products) ? me.purchased_products : []
+   const merged = Array.from(new Set([...fromBackend, ...knownProducts]))

    return {
-     email: me?.email || fallbackEmail,
+     email: me?.email || '',
      role: primaryRole,
      companyIds: ...,
      access,
-     purchasedStandards,
+     purchasedStandards: merged,
    }
  }
```

**Session-sjekk ved oppstart:**

```diff
  export function AuthProvider({ children }) {
-   const [authState, setAuthState] = useState(() => readStoredAuth())
+   const [authState, setAuthState] = useState(null)
+   const [loading, setLoading] = useState(true)
+
+   // Sjekk om det finnes en aktiv session ved oppstart
+   useEffect(() => {
+     fetch(`${API_BASE_URL}/me`, { credentials: 'include' })
+       .then((r) => (r.ok ? r.json() : null))
+       .then((me) => {
+         if (me) {
+           const stored = readStoredProducts(me.email)
+           setAuthState({ user: mapMeToUser(me, stored) })
+         }
+       })
+       .catch(() => {})
+       .finally(() => setLoading(false))
+   }, [])
```

**login — uten localStorage:**

```diff
  const login = async ({ email, password }) => {
    ...
    const me = await meRes.json()
-   const user = mapMeToUser(me, email)
-   const nextState = { user, token: null }
-   setAuthState(nextState)
-   writeStoredAuth(nextState)
+   const stored = readStoredProducts(me.email)
+   setAuthState({ user: mapMeToUser(me, stored) })
    return { ok: true }
  }
```

**logout — tømmer sessionStorage:**

```diff
  const logout = async () => {
    ...
+   clearStoredProducts()
    setAuthState(null)
-   writeStoredAuth(null)
  }
```

**buyStandard — frontend-state buffer for purchasedStandards:**

```diff
  const buyStandard = async ({ productCode, companyName, domainId, buyOption }) => {
    if (!domainId) {
      const buyFlowRes = await fetch(`${API_BASE_URL}/auth/buy-flow`, {
        body: JSON.stringify({ company_name: companyName }),
-                              buy_option: buyOption }),
      })
      if (!buyFlowRes.ok) return { ok: false }
    }

-   const buyProductBody = { product_code: productCode }
-   if (domainId) buyProductBody.domain_id = domainId
-   const buyProductRes = await fetch(`${API_BASE_URL}/auth/buy-product`, {
-     body: JSON.stringify(buyProductBody),
-   })
-   if (!buyProductRes.ok) return { ok: false }

    const meRes = await fetch(`${API_BASE_URL}/me`, { credentials: 'include' })
    if (!meRes.ok) return { ok: false }
    const me = await meRes.json()
-   const updatedUser = mapMeToUser(me, authState.user.email)
+   const updatedUser = mapMeToUser(me)

+   // Legg til produktkode i listen (frontend-state inntil backend har buy-product)
+   const prev = authState.user.purchasedStandards ?? []
+   const nextStandards = prev.includes(productCode) ? prev : [...prev, productCode]
+
+   writeStoredProducts(updatedUser.email, nextStandards)
    const nextState = {
-     ...authState,
-     user: updatedUser,
+     user: { ...updatedUser, purchasedStandards: nextStandards },
    }
    setAuthState(nextState)
-   writeStoredAuth(nextState)
    return { ok: true, role: nextState.user.role, purchasedStandards: nextStandards }
  }
```

**loading eksponert i kontekst:**

```diff
  const value = {
    user,
    isAuthenticated: Boolean(user),
+   loading,
    login, logout, buyStandard,
  }
```

**Forklaring:**
- `sessionStorage` tømmes automatisk når fanen lukkes — passer for midlertidig bufring av kjøpte standarder mens backend-endepunktet er under utvikling
- `Set` i `merged`-beregningen hindrer duplikater dersom både backend og sessionStorage inneholder samme kode
- `loading`-flag starter som `true` og settes til `false` etter at `/me`-sjekken er ferdig, uavhengig av utfall

---

## 2. ProtectedRoute — loading-guard

### Frontend/src/auth/ProtectedRoute.jsx (ENDRET)

```diff
  function ProtectedRoute({ children, allowedRoles = [], requiredStandard }) {
-   const { isAuthenticated, user } = useAuth()
+   const { isAuthenticated, user, loading } = useAuth()
+
+   if (loading) return null

    if (!isAuthenticated) {
      return <Navigate to="/login" replace />
    }
    ...
    if (requiredStandard) {
-     const purchasedStandards = Array.isArray(user.purchasedStandards)
-       ? user.purchasedStandards
-       : []
-     if (!purchasedStandards.includes(requiredStandard)) {
+     const purchased = Array.isArray(user.purchasedStandards) ? user.purchasedStandards : []
+     if (!purchased.includes(requiredStandard)) {
        return <Navigate to="/purchase" replace />
      }
    }
```

**Forklaring:**
- Uten `if (loading) return null` ville `isAuthenticated` vært `false` i de ~100ms `/me`-kallet tar, og brukeren ville bli sendt til `/login` selv med gyldig sesjon
- Returnerer `null` (ikke en spinner) for å unngå visuell flimring — siden er blank i den korte perioden

---

## 3. Flow-lagring: POST → PUT

### Frontend/src/auth/useFlowPage.js (ENDRET)

```diff
  const saveFlow = useCallback(async () => {
    if (!pageSlug || !domainId) return;
    await fetch(`${API_BASE_URL}/domains/${domainId}/flows/${pageSlug}`, {
-     method: 'POST',
+     method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      credentials: 'include',
      body: JSON.stringify({ page_slug: pageSlug, nodes, edges }),
    })
    setHasUnsavedChanges(false);
  }, [pageSlug, domainId, nodes, edges]);
```

---

## 4. FlowEdge-schema utvidet

### API/src/schemas/flows.py (ENDRET)

```diff
  class FlowEdge(BaseModel):
    sourceHandle: str | None = None
    targetHandle: str | None = None
    type: str = "straight"
+   style: dict | None = None
+   markerEnd: dict | None = None
```

**Forklaring:**
- React Flow sender `style` og `markerEnd` som en del av kant-objektet ved serialisering. Uten disse feltene i Pydantic-schemaet ville validering feile for kanter med tilpassede stiler eller pilhoder

---

## 5. FlowPageTemplate — fjernet underprosess-alternativ

### Frontend/src/pages/ISO-sider/9001-Undersider/Dynamiske/FlowPageTemplate.jsx (ENDRET)

```diff
  {[
    { key: 'label', text: 'Endre navn' },
    { key: 'description', text: 'Endre kort info' },
-   { key: 'slug', text: 'Sett som underprosess' },
  ].map((item) => ( ... ))}
```

**Forklaring:**
- "Sett som underprosess" ble lagt til 22. april, men krever backend-støtte for å fungere korrekt. Fjernet for å unngå å eksponere ufullstendig funksjonalitet.

---

## Oppsummering av endringer

- **5 filer endret**, netto -31 linjer (koden er kortere og mer ryddig)
- `AuthContext.jsx` er omstrukturert: ingen `localStorage`, session-sjekk ved oppstart, `loading`-flag
- `ProtectedRoute.jsx` venter på `loading` før den tar rutingbeslutninger
- `useFlowPage.js` bruker `PUT` for lagring
- `FlowEdge`-schema støtter nå `style` og `markerEnd`
- "Sett som underprosess" midlertidig fjernet fra redigeringsmenyen
