# Endringslogg - 10. februar 2026

## Fil endret
`src/pages/ISO-sider/ISO9001.jsx`

## Problem
Nodene (boksene) på ISO 9001-siden tilpasset seg ikke vindusstørrelsen. Ved endring fra fullskjerm til vindu ble noder kuttet av utenfor rammen. Noder på venstre og høyre side ble avskåret, og bare noen av de 9 nodene var synlige.

## Årsaker
1. `minZoom` og `maxZoom` var begge hardkodet til `0.7`, som låste zoomen helt og hindret `fitView` fra å skalere innholdet til å passe i viewporten
2. Selv etter at den hardkodede zoom-låsen ble fjernet, var React Flow sin standard `minZoom` (0.5) fortsatt for høy — nodene strekker seg ~2350px horisontalt, og trenger en zoom på ca. 0.3 for å passe i et smalt vindu
3. Ingen resize-lytter — `fitView` kjørte bare ved første render, så ved endring av vindusstørrelse (f.eks. fullskjerm til vindu) ble ikke nodene repositionert

---

## Detaljerte endringer

### 1. Import utvidet med `useEffect`

**Lagt til:** `useEffect` i React-importen for å kunne lytte på vindu-endringer.

```jsx
// Før
import { useState, useCallback } from 'react';

// Etter
import { useState, useCallback, useEffect } from 'react';
```

### 2. Ny state: `rfInstance`

**Lagt til:** En ny state-variabel for å lagre referansen til React Flow-instansen. Denne brukes til å kalle `fitView()` programmatisk etter første render.

```jsx
const [rfInstance, setRfInstance] = useState(null);
```

### 3. Ny `useEffect`: Resize-lytter

**Lagt til:** En `useEffect` som registrerer en event listener på `window` sitt `resize`-event. Når vinduet endrer størrelse, kalles `rfInstance.fitView({ padding: 0.2 })` som automatisk skalerer og sentrerer alle noder innenfor den tilgjengelige plassen. Cleanup-funksjonen fjerner listeneren ved unmount.

```jsx
useEffect(() => {
  if (!rfInstance) return;
  const handleResize = () => rfInstance.fitView({ padding: 0.2 });
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, [rfInstance]);
```

### 4. Fjernet: Låst zoom (`minZoom={0.7}` / `maxZoom={0.7}`)

**Fjernet:** De to props-ene `minZoom={0.7}` og `maxZoom={0.7}` på `<ReactFlow>`-komponenten. Disse låste zoom-nivået til nøyaktig 0.7, noe som forhindret `fitView` fra å justere zoom fritt. Uten disse kan React Flow nå velge det zoom-nivået som passer best for gjeldende vindusstørrelse.

```jsx
// Fjernet disse to linjene:
minZoom={0.7}
maxZoom={0.7}
```

### 5. Lagt til: `minZoom={0.1}`

**Lagt til:** `minZoom={0.1}` som ny prop. React Flow sin standard `minZoom` er 0.5, men med 9 noder spredt over ~2350px horisontalt trenger `fitView` å kunne zoome ned til ca. 0.3 for å vise alt på mindre skjermer. Ved å sette `minZoom={0.1}` har `fitView` nok spillerom.

```jsx
minZoom={0.1}
```

### 6. Lagt til: `onInit={setRfInstance}`

**Lagt til:** `onInit`-propen på `<ReactFlow>`. Denne callbacken kalles av React Flow når komponenten er ferdig initialisert, og gir oss tilgang til instansen (med metoder som `fitView`, `getZoom`, etc.). Vi lagrer den i `rfInstance`-staten slik at resize-lytteren kan bruke den.

```jsx
onInit={setRfInstance}
```

### 7. Lagt til: `fitViewOptions={{ padding: 0.2 }}`

**Lagt til:** `fitViewOptions` som prop sammen med det eksisterende `fitView`-proppen. `padding: 0.2` gir 20% ekstra plass rundt nodene slik at de ikke klistrer seg helt inntil kanten av rammen.

```jsx
fitView
fitViewOptions={{ padding: 0.2 }}
```

---

## Oppsummering av props på `<ReactFlow>` etter endring

| Prop | Før | Etter | Forklaring |
|------|-----|-------|------------|
| `minZoom` | `0.7` (låst) | `0.1` | Tillater mye mer utzoom |
| `maxZoom` | `0.7` (låst) | *(fjernet, bruker standard 1.5)* | Ikke lenger låst |
| `onInit` | *(fantes ikke)* | `{setRfInstance}` | Fanger React Flow-instansen |
| `fitViewOptions` | *(fantes ikke)* | `{{ padding: 0.2 }}` | 20% padding rundt nodene |
| `fitView` | `true` | `true` | Uendret |
| `panOnDrag` | `false` | `false` | Uendret |
| `panOnScroll` | `false` | `false` | Uendret |

## Resultat
Alle 9 noder passer nå innenfor rammen uansett vindusstørrelse, og tilpasser seg automatisk ved resize. Løsningen fungerer på alt fra fullskjerm til lite vindu.
