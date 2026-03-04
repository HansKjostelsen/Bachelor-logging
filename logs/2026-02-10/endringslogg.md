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

---

## Begrunnelse

Valgene som ble tatt denne dagen var drevet av et grunnleggende brukervennlighetskrav: et prosessdiagram som kuttes av eller krever manuell zooming mister mye av verdien sin. Siden FlowCRT er ment å gi brukere en rask og tydelig oversikt over ISO 9001-prosessene, var det viktig at hele diagrammet alltid er synlig uten ekstra innsats fra brukeren.

Valget om å bruke `useEffect` med en `resize`-lytter fremfor en ren CSS-løsning var fornuftig fordi React Flow opererer med sitt eget interne koordinatsystem. Det er ikke mulig å tvinge nodene inn i riktig visning utelukkende via CSS — `fitView()` må kalles programmatisk på React Flow-instansen. `onInit`-propen er den anbefalte måten å få tak i denne instansen på, og kombinasjonen med `useState` gjør at lytteren alltid har tilgang til en gyldig instans.

Å sette `minZoom={0.3}` i stedet for et lavere tall som `0.1` var et bevisst valg basert på testing. For lav `minZoom` gir brukeren mulighet til å zoome så langt ut at nodene blir uleselige og mister all informasjonsverdi. `0.3` ble funnet som et praktisk minimum der alle noder fortsatt er lesbare på typiske skjermstørrelser.

`translateExtent` ble innført for å løse det klassiske problemet med uendelig canvas: brukere kan ved uhell pane bort fra innholdet og bli desorientert. Ved å sette grenser med litt padding rundt nodene beholdes navigasjonsfriheten, men det blir umulig å "miste" seg bort fra diagrammet. Dette er spesielt viktig i en bacheloroppgave-kontekst der sluttbrukere ikke nødvendigvis er vant til interaktive flytdiagrammer.

CSS-endringen fra fast `height: 600px` til `height: calc(100vh - 120px)` på `.flow-wrapper` var nødvendig for at diagrammet skal utnytte tilgjengelig skjermhøyde uavhengig av vindusstørrelse. En hardkodet pikselhøyde ville gitt dårlig opplevelse på både store og små skjermer, og motvirket hele poenget med de øvrige responsive tilpasningene.

---

## Ytterligere forbedring: Begrenset pan-område med `translateExtent`

**Dato:** 10. februar 2026 (senere samme dag)

### Problem
Selv etter at nodene ble responsive og tilpasset vindusstørrelsen, kunne brukeren fortsatt panorere uendelig langt bort fra nodene i alle retninger. Dette gjorde det lett å "miste" innholdet og måtte zoome ut eller refreshe for å finne nodene igjen.

### Løsning
Lagt til `translateExtent`-propen på `<ReactFlow>`-komponenten for å begrense pan-området.

### Implementasjon

**Lagt til:** `translateExtent` prop med verdien `[[-200, -600], [2600, 650]]`.

```jsx
translateExtent={[
  [-200, -600],
  [2600, 650],
]}
```

**Beregning av verdiene:**
- Nodene strekker seg fra x=0 til x≈2350 (node n7 på x=2100 + nodebredde ~250)
- Nodene strekker seg fra y=-350 (node n8) til y=400 (node n9) + nodehøyde ~50
- Lagt til 200px padding på venstre side (x-minimum: -200)
- Lagt til 250px padding på høyre side (x-maksimum: 2600)
- Lagt til 250px padding på toppen (y-minimum: -600)
- Lagt til 200px padding på bunnen (y-maksimum: 650)

**Rydding:**
- Fjernet utkommentert `panOnDrag={false}` og `panOnScroll={false}` (disse var kommentert ut fra tidligere)
- Fjernet utkommentert `nodeExtent={[[0, 0], [1000, 600]]}` (gammel kode som ikke lenger var relevant)

**Endret:** `minZoom` fra `0.1` til `0.3`.
Etter testing viste det seg at `minZoom={0.3}` er tilstrekkelig for at alle noder skal passe i viewport på typiske skjermstørrelser. `0.1` var for konservativt og tillot unødvendig mye utzoom.

```jsx
// Før
minZoom={0.1}

// Etter
minZoom={0.3}
```

### Resultat
Brukeren kan nå kun panorere innenfor et begrenset område rundt nodene, med fornuftig padding på alle sider. Dette gjør det umulig å "miste" innholdet ved utilsiktet panning, samtidig som det gir nok rom til å navigere komfortabelt rundt flowchartet.
