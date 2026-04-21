# Kodeendringer - 2026-04-21

Dette dokumentet viser de faktiske kodeendringene som ble gjort 21. april 2026 i FlowCRT-prosjektet.

**Diff-område:** Ucommitted endringer i 3 frontend-filer

**Merk:** Kun frontend-endringer denne dagen. Fokus på steg-wizard for kjøpssiden.

---

## Hovedendringer

### 1. buyStandard oppdatert til objekt-parameter

#### Frontend/src/auth/AuthContext.jsx (ENDRET)

```diff
-  const buyStandard = async (standardCode, companyName) => {
+  const buyStandard = async ({ productCode, companyName, domainId }) => {
     if (!authState?.user) {
       return { ok: false, error: 'Not authenticated' }
     }

     try {
-      const isVisitor = authState.user.role === ROLES.VISITOR
-
-      if (isVisitor) {
+      if (!domainId) {
         const buyFlowRes = await fetch(`${API_BASE_URL}/auth/buy-flow`, {
           method: 'POST',
           headers: { 'Content-Type': 'application/json' },
           credentials: 'include',
-          body: JSON.stringify({ company_name: companyName || standardCode }),
+          body: JSON.stringify({ company_name: companyName }),
         })
         if (!buyFlowRes.ok) return { ok: false }
       }

+      const buyProductBody = { product_code: productCode }
+      if (domainId) buyProductBody.domain_id = domainId
+
       const buyProductRes = await fetch(`${API_BASE_URL}/auth/buy-product`, {
         method: 'POST',
         headers: { 'Content-Type': 'application/json' },
         credentials: 'include',
-        body: JSON.stringify({ product_code: standardCode }),
+        body: JSON.stringify(buyProductBody),
       })
       if (!buyProductRes.ok) return { ok: false }
```

**Forklaring:**
- Betingelsen for å kalle `/auth/buy-flow` er endret fra å sjekke `VISITOR`-rolle til å sjekke om `domainId` mangler — dette er mer generelt og støtter at en FLOW_ADMIN også kan opprette nye bedrifter
- `buyProductBody` bygges dynamisk: `domain_id` inkluderes kun dersom det er satt, slik at backend kan vite hvilken bedrift kjøpet skal knyttes til
- Fallback `companyName || standardCode` er fjernet — bedriftsnavnet er nå alltid eksplisitt oppgitt av brukeren via wizard-steget

---

### 2. PurchaseStandards omskrevet til steg-wizard

#### Frontend/src/pages/PurchaseStandards.jsx (FULL OMSKRIVING)

**State-maskin:**

```jsx
const [step, setStep] = useState('company')          // 'company' | 'product' | 'done'
const [companyName, setCompanyName] = useState('')
const [selectedDomainId, setSelectedDomainId] = useState(null)
const [companyMode, setCompanyMode] = useState('existing')  // 'existing' | 'new'
const [purchasedProduct, setPurchasedProduct] = useState(null)
const [displayCompanyName, setDisplayCompanyName] = useState('')
```

**Steg 1 — Bedriftsvalg (VISITOR):**

```jsx
{isVisitor ? (
  <>
    <p>Skriv inn bedriftsnavnet ditt. Ved første kjøp blir kontoen oppgradert til
      <strong> Flow administrator (FLOW_ADMIN)</strong>.
    </p>
    <label className="purchase-company-label">
      Bedriftsnavn
      <input
        type="text"
        className="purchase-company-input"
        value={companyName}
        onChange={(e) => setCompanyName(e.target.value)}
        placeholder="Skriv inn bedriftsnavn..."
      />
    </label>
  </>
) : ( /* FLOW_ADMIN dropdown */ )}
```

**Steg 1 — Bedriftsvalg (FLOW_ADMIN/SYSTEM_ADMIN):**

```jsx
<select
  className="purchase-company-select"
  value={companyMode === 'new' ? '__new__' : (selectedDomainId ?? '')}
  onChange={handleDomainSelectChange}
>
  <option value="" disabled>-- Velg bedrift --</option>
  {existingDomains.map((d) => (
    <option key={d.domain_id} value={d.domain_id}>
      {d.company_name || `Bedrift #${d.domain_id}`}
    </option>
  ))}
  <option value="__new__">+ Ny bedrift</option>
</select>

{companyMode === 'new' && (
  <label className="purchase-company-label">
    Nytt bedriftsnavn
    <input type="text" className="purchase-company-input" ... />
  </label>
)}
```

**handleDomainSelectChange:**

```jsx
const handleDomainSelectChange = (e) => {
  const value = e.target.value
  if (value === '__new__') {
    setCompanyMode('new')
    setSelectedDomainId(null)
  } else {
    setCompanyMode('existing')
    setSelectedDomainId(Number(value))
  }
}
```

**Steg 2 — Produktvalg:**

```jsx
{step === 'product' && (
  <div className="purchase-wizard-step">
    <h3>Velg produkt for {displayCompanyName}</h3>
    <div className="purchase-grid">
      {STANDARD_CATALOG.map((standard) => {
        const isPurchased = purchasedStandards.includes(standard.code)
        return (
          <div className={`purchase-card ${isPurchased ? 'purchased' : ''}`}>
            <h3>{standard.title}</h3>
            <button onClick={() => handleBuy(standard)} disabled={isPurchased}>
              {isPurchased ? 'Allerede kjøpt' : 'Kjøp'}
            </button>
          </div>
        )
      })}
    </div>
  </div>
)}
```

**Steg 3 — Bekreftelse:**

```jsx
{step === 'done' && (
  <div className="purchase-wizard-step purchase-confirmation">
    <h3>Kjøp registrert!</h3>
    <p><strong>{purchasedProduct?.title}</strong> er registrert for <strong>{displayCompanyName}</strong>.</p>
    <p><strong>Aktiv rolle:</strong> {ROLE_LABELS[user?.role] || user?.role}</p>
    <div className="purchase-step-actions">
      <button onClick={handleBuyMore} className="purchase-btn-secondary">Kjøp mer</button>
      <button onClick={() => navigate('/iso9001')} className="purchase-btn-primary">Gå til diagram</button>
    </div>
  </div>
)}
```

**handleBuy-logikken:**

```jsx
const handleBuy = async (standard) => {
  const payload = { productCode: standard.code }
  if (selectedDomainId && companyMode === 'existing') {
    payload.domainId = selectedDomainId
  } else {
    payload.companyName = companyName.trim()
  }
  const result = await buyStandard(payload)
  if (!result.ok) { setMessage('Kjøp feilet.'); return }
  setPurchasedProduct(standard)
  setStep('done')
}
```

**Forklaring:**
- Wizard-state styres av `step`-variabelen som har tre mulige verdier: `company`, `product` og `done`
- `companyMode` skiller mellom "velg eksisterende" og "opprett ny" for FLOW_ADMIN — dropdown-verdien `__new__` trigger ny-modus
- `displayCompanyName` settes ved overgang fra steg 1 til 2, slik at bedriftsnavnet vises korrekt i steg 2 og 3 uavhengig av modus
- `canProceedFromCompany` validerer at brukeren har fylt inn nødvendig informasjon før "Neste"-knappen aktiveres
- `handleBuyMore` nullstiller all wizard-state og tar brukeren tilbake til steg 1

---

### 3. CSS for wizard-komponenter

#### Frontend/src/styles.css (ENDRET)

```css
/* Steg-indikator */
.purchase-step-indicator {
  display: flex;
  justify-content: center;
  gap: 2rem;
  margin: 1.5rem 0 0.5rem;
}

.purchase-step-number {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: #e0e0e0;
  color: #666;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
}

.purchase-step-dot.active .purchase-step-number {
  background: #013220;
  color: #fff;
}

.purchase-step-dot.completed .purchase-step-number {
  background: #145c25;
  color: #fff;
}

/* Input og select */
.purchase-company-input,
.purchase-company-select {
  padding: 0.9rem 1rem;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 1rem;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.purchase-company-input:focus,
.purchase-company-select:focus {
  border-color: #013220;
  box-shadow: 0 0 0 3px rgba(1, 50, 32, 0.1);
}

/* Knapper */
.purchase-btn-primary {
  background: #013220;
  color: white;
  border: none;
  padding: 0.85rem 2rem;
  font-weight: 600;
  border-radius: 8px;
}

.purchase-btn-primary:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.purchase-btn-secondary {
  background: #fff;
  color: #013220;
  border: 2px solid #013220;
  padding: 0.85rem 2rem;
  font-weight: 600;
  border-radius: 8px;
}

/* Bekreftelse */
.purchase-confirmation { text-align: center; }
.purchase-card.purchased { opacity: 0.6; }
```

**Forklaring:**
- Fargene er konsistente med FlowCRT-paletten: `#013220` (primær), `#145c25` (hover/completed), `#e0e0e0` (inaktiv)
- Steg-indikatoren bruker `border-radius: 50%` for sirkulære steg-numre med tre tilstander: inaktiv (grå), aktiv (mørkegrønn), fullført (lysere grønn med hakesymbol)
- Input-stilen matcher eksisterende `.login-form input` og `.profile-form input` for visuell konsistens
- `.purchase-card.purchased` med `opacity: 0.6` gir tydelig visuell indikasjon på at et produkt allerede er kjøpt

---

## Oppsummering av endringer

- **3 filer endret**, ca. 200 linjer lagt til, 50 linjer slettet
- `buyStandard` støtter nå to flyter: ny bedrift (kaller `/auth/buy-flow` + `/auth/buy-product`) og eksisterende bedrift (kun `/auth/buy-product` med `domain_id`)
- PurchaseStandards er omskrevet fra flatt grid til 3-stegs wizard med bedriftsvalg, produktvalg og bekreftelse
- CSS-endringene følger eksisterende designsystem med FlowCRT-farger og inputstiler
