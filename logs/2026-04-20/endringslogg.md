# Endringslogg - Søndag 20. april 2026

## Oversikt
Implementerte innsetting av noder mellom eksisterende noder (`insertNodeOnEdge`) og `hasUnsavedChanges`-mekanisme i `useFlowPage.js`. I tillegg ble `react-helmet-async` installert og lockfilen ryddet opp.

---

## Commits (kronologisk)

### 1. Innsetting av node mellom eksisterende noder og hasUnsavedChanges (`0bbc3444`)

**Tidspunkt:** 13:27

**Endret i `Frontend/src/auth/useFlowPage.js`:**
- Lagt til `insertNodeOnEdge`-funksjon: velg en edge, klikk "Sett inn" — en ny node plasseres i midtpunktet mellom source og target, og target forskyves med halve avstanden slik at gapet bevares
- Handles på den nye noden bestemmes automatisk basert på den originale edgens `sourceHandle` via et `oppositeHandle`-oppslag
- Den gamle edgen erstattes med to nye: source→ny node og ny node→target
- Lagt til `selectedEdge`-state og `onEdgeClick`/`onPaneClick`-callbacks for edge-seleksjon
- Lagt til `hasUnsavedChanges`-state som settes til `true` ved alle meningsfulle endringer (filtrerer ut `select` og `dimensions` fra `onNodesChange`) og tilbakestilles etter lagring

**Endret i `Frontend/src/pages/9001-Undersider/Dynamiske/FlowPageTemplate.jsx`:**
- Koblet opp `onEdgeClick`, `onPaneClick`, `selectedEdge`, `hasUnsavedChanges` og `insertNodeOnEdge` fra `useFlowPage`
- "Sett inn"-knapp i toolbar som er aktiv kun når en edge er valgt
- Lagre-knapp deaktiveres (`disabled`) når `hasUnsavedChanges` er `false`

### 2. Installer react-helmet-async (`80244799`)

**Tidspunkt:** 13:29

**Endret i `Frontend/package-lock.json`:**
- Ryddet opp overflødige entries i lockfilen (11 slettede linjer)

---

## Oppsummering
- **2 filer endret**, ca. 144 linjer lagt til, 6 linjer slettet
- Ny frontend-funksjonalitet: insertNodeOnEdge for innsetting av noder mellom eksisterende noder i flowdiagrammer
- hasUnsavedChanges gir visuell tilbakemelding på lagringsstatus — Lagre-knappen er deaktivert når det ikke er endringer

---

## Begrunnelse

### Hvorfor insertNodeOnEdge med midtpunktsplassering?

Alternativet hadde vært å plassere den nye noden på en fast posisjon og la brukeren dra den manuelt. Ved å beregne midtpunktet mellom source og target, og forskyve target med halve avstanden, bevares den visuelle flyten i diagrammet automatisk. Brukeren slipper manuell justering og kan umiddelbart begynne å redigere den nye noden. Denne tilnærmingen er inspirert av hvordan Figma og lignende verktøy håndterer innsetting mellom elementer.

### Hvorfor filtrere bort select/dimensions i hasUnsavedChanges?

React Flow sender `onNodesChange` for mange typer endringer, inkludert `select` (klikk på node) og `dimensions` (noden rendres/resizes). Disse representerer ikke reelle brukerendringer som bør markeres som ulagrede. Uten filtreringen ville et enkelt klikk på en node trigge "ulagrede endringer", noe som er forvirrende for brukeren.

---

## Ikke-koderelatert arbeid

I tillegg til kodeendringene ble det brukt tid på:

1. **Manuell testing av insertNodeOnEdge** — verifiserte at noden plasseres korrekt mellom source og target, at handles kobles riktig, og at target forskyves som forventet
2. **Testing av hasUnsavedChanges** — sjekket at Lagre-knappen aktiveres ved reelle endringer og ikke ved rene seleksjoner
