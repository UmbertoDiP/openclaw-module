---
system: OpenClaw
kind: living-context
---

# CONTEXT

## Obiettivo

Consolidare il “Sacro Graal” (master prompt/visione) di OpenClaw in 3 file master per un AI builder: zero codice in questa fase, requisiti deterministici, flussi operativi e modelli dati coerenti.

## Fonte di verità (ordine di lettura)

1. `openclaw/1_PRD_Master.md`
2. `openclaw/2_UI_Architecture_and_Flows.md`
3. `openclaw/3_Data_Models_and_Mock_Data.md`

Regola: se un’informazione è necessaria per costruire il modulo/app, deve stare in uno di questi tre file (evitare frammentazione).

## Sacro Graal (protezione)

- Fonte originale (editabile): `openclaw/0_CORE_DA_COMPILARE.md`
- Backup immutabile (read-only): `openclaw/0_CORE__SACRO_GRAAL__IMMUTABLE__20260518_215205.md`

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

- Derivare requisiti, flussi e modelli dati dal Sacro Graal, mantenendo “DA DECIDERE” dove manca una scelta esplicita.
