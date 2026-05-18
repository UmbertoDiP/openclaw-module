---
system: OpenClaw
kind: living-context
---

# CONTEXT

## Obiettivo

Creare un modulo “OpenClaw” (contesto-only) progettato per essere consumato da un AI builder: poche fonti master, zero codice in questa fase, focus su business logic e superfici UI necessarie.

## Fonte di verità (ordine di lettura)

1. `openclaw/1_PRD_Master.md`
2. `openclaw/2_UI_Architecture_and_Flows.md`
3. `openclaw/3_Data_Models_and_Mock_Data.md`

Regola: se un’informazione è necessaria per costruire il modulo/app, deve stare in uno di questi tre file (evitare frammentazione).

## Stato Corrente

- Repo inizializzato come template “OpenClaw module” (solo Markdown).
- Struttura consolidata in 3 file master.

## Convenzioni

- Nessun codice applicativo.
- Ambiguità: “DA DECIDERE” + domanda esplicita.
- Business logic: regole deterministiche (quando/se → allora) + eccezioni.
- UI: per ogni vista indicare stati (loading/empty/success/error) e azioni.
- Data: includere mock JSON realistici e coerenti con la UI.

## Definition of Done (per input AI builder)

- PRD con scope/non-scope, requisiti e acceptance criteria.
- UI: viste, navigazione, componenti principali, stati.
- Dati: modelli, relazioni, mock JSON (singolo/lista) + payload errori.

## Next Step

- Riempire i 3 master file a partire dal “santo graal” del modulo OpenClaw.
