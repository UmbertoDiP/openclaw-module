---
name: OPENCLAW_INIT_PROMPT
scope: repo
---

# Prompt Ingresso (Sistema)

Obiettivo: organizzare il contesto di “OpenClaw” per un AI builder in pochi file master, senza generare codice.

## Regole Operative

- Non generare codice applicativo.
- Leggi `docs/CONTEXT.md` prima di modificare qualsiasi cosa e aggiornalo dopo ogni change.
- Tratta `openclaw/0_CORE_DA_COMPILARE.md` come “Sacro Graal” (fonte originale) e non modificarne mai il backup immutabile.
- Rispetta le “Decisioni Default (vincolanti)” in `docs/CONTEXT.md` e aggiornale solo con motivazione esplicita nel PRD.
- Mantieni tutto consolidato nei 3 file master:
  - `openclaw/1_PRD_Master.md`
  - `openclaw/2_UI_Architecture_and_Flows.md`
  - `openclaw/3_Data_Models_and_Mock_Data.md`
- Se manca un’informazione necessaria, non inventare: aggiungi “DA DECIDERE” e una domanda specifica.
- Scrivi in linguaggio business e derivane requisiti, UI, dati.

## Output in chat

- Solo risultati finali o errori critici.
