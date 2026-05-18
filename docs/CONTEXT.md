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

## Decisioni Default (vincolanti)

- Interfaccia AI: Open WebUI.
- Vault/Secrets: HashiCorp Vault (single-node per VM) + redazione obbligatoria.
- Vector RAG: Postgres + pgvector (locale VM) con isolamento per tenant.
- Audit log: append-only + export bundle; retention 365 giorni (configurabile).
- Host → VM: solo 22 (SSH) e 8080 (reverse proxy).
- Subnet per tenant: 172.29.0.0/16, allocazione /24 per tenant.
- Naming: `tnt_<slug>`, `node_<slug>_<nn>`, `run_<yyyymmdd>_<seq>`.

## Stato Corrente

- Repo inizializzato come template “OpenClaw module” (solo Markdown).
- Sacro Graal protetto (fonte editabile + backup immutabile read-only).
- Decisioni default fissate (UI/Vault/RAG/Audit/porte/subnet/naming).
- Struttura consolidata e coerente in 3 file master (PRD/UI/Data), senza “DA DECIDERE” nei master.
- Backlog di implementazione (Jira) derivato dai FR nel PRD.
- Snapshot Git disponibili (tag):
  - sacred-grail-20260518_215205
  - decisions-default_20260518_220306
  - models-mocks_20260518_220743
  - kpi-invariants_20260518_220933
  - traceability_20260518_221109
  - ui-traceability_20260518_221346
  - jira-backlog_20260518_221754

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

- Handoff all’AI builder: usare i 3 master come unica fonte di verità per avviare la fase di implementazione (in un workspace/app dedicato).

## Fase Codice (pre-clone → start coding)

### Freeze & governance

- Usa come “contratto” immutabile: `openclaw/1_PRD_Master.md`, `openclaw/2_UI_Architecture_and_Flows.md`, `openclaw/3_Data_Models_and_Mock_Data.md` + questo file.
- Ogni deviazione da Decisioni Default/KPI/Invarianti/Matrici richiede ADR esplicito e tag Git dedicato.

### Workspace strategy (“Fork, Merge & Inject”)

- Crea una workspace separata per la fase codice (non in questo repo contesto-only).
- Importa il contesto come input read-only nella nuova workspace; i master restano la fonte (eventuale copia solo per consultazione).
- Se l’ambiente limita operazioni fuori workspace: usa un submodule “upstream” dentro questo repo solo per scaricare/agganciare il codice sorgente.

### Prerequisiti tecnici

- Repo ufficiale OpenClaw: https://github.com/openclaw/openclaw (pinned: `v2026.4.26` / commit `be8c246`).
- Workspace fase codice (default): una cartella separata accanto a questo repo (es. `../openclaw-code`).
- Definisci repo ufficiale OpenClaw e modalità: fork vs clone diretto (preferire fork per isolare le modifiche).
- Policy segreti: nessun `.env` committato; segreti solo via secret manager; redazione log obbligatoria.
- Baseline verifica: lint/test, CI locale, standard commit/tag per step funzionali.

### Clone/bootstrap

- Clona/forka la repo ufficiale nella nuova workspace e crea branch `integration-from-context`.
- Inietta il contratto: mappa FR → moduli/servizi reali e crea backlog tecnico equivalente.
- Se usi il submodule in questo repo: il codice upstream è agganciato in `upstream/openclaw` (checkout `v2026.4.26`).

### Gap analysis (repo vs contratto)

- Inventory componenti esistenti (auth, orchestrazione, audit, RAG, finops, proxy, provisioning, offboarding).
- Per ogni FR: “già coperto / parziale / mancante” usando la matrice FR↔UI↔Data↔Audit del PRD.
- Lista “missing code units” (moduli, API, job, workflow n8n, policy enforcement, export bundle) con priorità.

### Implementazione per tranche (audit-first)

- Tranche A: AuditEvent + correlazione + redazione + export bundle.
- Tranche B: Jira↔n8n pipeline + guardrail Pending Review (policy block).
- Tranche C: Vault/Secrets contract + enforcement “no leak”.
- Tranche D: RAG (Confluence cache → pgvector) + sync schedulato.
- Tranche E: FinOps (TokenUsage/BudgetLimit) + watchdog/kill-switch.
- Tranche F: Snapshot/rollback + offboarding.
- Tranche G: Reverse proxy routing (single ingress) e hardening.

### Verifica & release discipline

- Per ogni tranche: test minimi + evidenze (log redatti) + tag Git.
- Verifica KPI e invarianti (tenant isolation, no secrets, pending review hard stop).
