---
title: PRD Master — OpenClaw
audience: AI Builder
version: 0.1
---

# 1) PRD Master — OpenClaw

## 1.1 Panoramica

- Nome: OpenClaw
- Descrizione (1 riga): NexusNode — “Golden Image” Hyper-V per eseguire OpenClaw in modo isolato per-cliente, con stack Docker, audit e guardrail operativi.
- Problema che risolve: dare a un agente AI operativo un ambiente per-cliente sicuro, tracciabile e reversibile, evitando contaminazioni tra clienti e azioni irreversibili in produzione.
- Risultato atteso per l’utente: clonare un nodo cliente in modo ripetibile, far lavorare l’agente su task tracciati (Jira) con audit end-to-end, e mantenere controllo (Pending Review, rollback, kill-switch, offboarding).

## 1.1A KPI / Criteri di Successo

- Provisioning: un Client Node raggiunge stato READY e risponde a SSH entro 15 minuti dalla richiesta.
- Isolamento rete: 0 casi di reachability inter-VM/LAN per un nodo cliente; egress Internet operativo per 80/443.
- Workflow Jira: 100% dei ticket “Selected for AI” terminano automaticamente in “Pending Review” (mai oltre).
- Audit: 100% dei run generano eventi audit correlati (tenantId + ticketId + runId) e export bundle riuscito.
- Reversibilità: ripristino da snapshot DB con RTO ≤ 5 minuti (per il DB target) e con evidenza audit.
- Kill-switch: runaway CPU/MEM o runaway calls determinano STOP_RUN o DISABLE_AGENT entro 60 secondi dal superamento soglia.
- Confluence cache/RAG: sync schedulato con success rate ≥ 99% e retrieval locale senza dipendenza runtime dalla rete.
- FinOps: budget breach genera blocco esecuzioni (STOP_RUNS) e audit event; nessuna esecuzione oltre budget senza override Operatore.

## 1.2 Scope / Non-Scope

**In scope**
- Golden Image Hyper-V clonabile per cliente (“Client Node”).
- Provisioning da host Windows: clonazione VM + isolamento rete (Isolated NAT + ACL su vNIC) + accesso da host (SSH e web via reverse proxy).
- Stack base in VM (Docker): orchestrazione automazione (n8n), interfaccia AI (Open WebUI), componenti per automazione browser headless e debugging remoto.
- Integrazione operativa Jira ↔ n8n ↔ OpenClaw con state machine inviolabile fino a Pending Review.
- Sync Confluence → cache Markdown locale per RAG + indicizzazione vettoriale locale (Vector RAG).
- 12 moduli infrastrutturali: Security/Vault, Audit/Logging, Local CI runner, Reverse Proxy, Snapshot/Rollback DB, Kill-switch, Onboarding wizard, Vector RAG, Air-Lock VPN, Agent Toolbelt API, FinOps/Osservabilità, Offboarding/Cold storage.
- Protocolli: identità agente (SOUL) e policy Jira/Audit (JIRA_AUDIT_PROTOCOL).

**Non-scope**
- Implementazione di funzionalità business specifiche del cliente (applicazioni cliente).
- Gestione multi-host / clusterizzazione di più host Hyper-V.
- Sostituzione di Jira/Confluence: il sistema integra e automatizza, non rimpiazza.
- Deploy diretto in produzione senza revisione umana (vietato per requisito).

## 1.2A Strategia Upstream (OpenClaw) & Wrapper

- Obiettivo: assemblare un prodotto unico mantenendo OpenClaw come upstream separato e aggiornabile; tutte le personalizzazioni vivono nel wrapper.
- Vincolo: evitare fork permanenti dell’upstream; se serve una modifica generica, aprire PR upstream. Il wrapper gestisce integrazione/contratto/guardrail.
- Struttura:
  - Upstream: repo OpenClaw ufficiale come submodule (pinned a una release/tag).
  - Wrapper (“wrap system”): componenti OpenClaw-Module che orchestrano VM, guardrail, audit, policy Jira, e integrazioni (n8n/Confluence/Vault), esponendo un contratto stabile.
- Strategia upgrade:
  - Step 1: aggiornare il puntamento del submodule a una release/tag successiva.
  - Step 2: eseguire una gap analysis dei punti di integrazione (API/CLI/config/plugin).
  - Step 3: adeguare il wrapper e aggiornare test/diagnostica.
  - Step 4: taggare la release del wrapper, mantenendo tracciabilità (versione upstream + commit).

### 1.2A.1 Boundary tecnico (STORY-00.2) — entrypoint upstream e responsabilità wrapper

Principio: il wrapper usa upstream come “engine” tramite superfici stabili (CLI/RPC/config/plugins) e aggiunge governance/tenancy/audit/policy; non modifica codice upstream nel repository wrapper.

**Superfici upstream ammesse (entrypoint)**
- CLI `openclaw …`: invocazione comandi per operazioni standard (status/health/logs/secrets/memory/export-trajectory).
- Gateway API/RPC (WebSocket): uso di metodi esposti dal Gateway per interrogazioni/azioni “remote-safe” (es. tail logs, reload secrets).
- Configurazione: gestione `openclaw.json` e snapshot runtime secondo convenzioni upstream (no campi custom non supportati).
- Plugin/skills: uso delle estensioni upstream come unità caricabili; nessuna patch locale del codice upstream.

**Responsabilità del wrapper (non upstream)**
- Tenancy e lifecycle: `Tenant`/`Node` (Hyper‑V) + provisioning + onboarding + offboarding.
- Orchestrazione Jira/Confluence/n8n: state machine inviolabile fino a Pending Review e correlazione con ticket/run.
- Audit “business”: `AuditEvent` append-only con correlazione `tenantId/ticketId/runId`, redazione, explorer, export bundle contrattuale.
- Security/Vault: enforcement “no secrets in chiaro”, accesso minimo necessario, policy + audit su accessi.
- Reverse proxy: single ingress host 8080, routing per-tenant, health, audit cambi.
- FinOps/Osservabilità: tracking e budget enforcement (BUDGET_BREACH → STOP_RUNS) con audit.
- DB snapshot/rollback e watchdog: enforcement policy (SNAPSHOT_REQUIRED, STOP_RUN/DISABLE_AGENT) con eventi correlati.

### 1.2A.2 Contratto wrapper ↔ upstream (v1)

| Area | Wrapper fornisce | Wrapper chiama upstream via | Note vincolanti |
|---|---|---|---|
| Identità run | `runId` deterministico + binding a tenant/ticket | CLI/Gateway (come “engine”) | Ogni invocazione upstream deve ereditare `runId` e produrre audit correlato nel wrapper |
| Audit/diagnostica | `AuditEvent` + export bundle per tenant/ticket/run | `openclaw logs`, `export-trajectory` | Export upstream è “support bundle”; non sostituisce export contrattuale wrapper |
| Secrets | Vault/SecretItem + redazione obbligatoria | `openclaw secrets …` | Wrapper gestisce policy e audit; upstream gestisce SecretRefs/snapshot runtime |
| Memoria/RAG | Confluence cache + pgvector per-tenant | `openclaw memory …` (se usato) | L’isolamento per-tenant e la provenance Confluence sono responsabilità wrapper |
| Guardrail operativi | Pending Review hard-block, kill-switch, budget stop | `/kill` e gestione run (se disponibile) | Il wrapper è la fonte di verità per policy; upstream non deve “bypassare” il blocco |

### 1.2A.3 Regole di compatibilità (upgrade-safe)

- Il wrapper non dipende da dettagli interni di upstream (file path, shape di storage non documentati); dipende solo da entrypoint pubblici (CLI/docs/contracts).
- Ogni bump upstream richiede: aggiornamento `UpstreamSourcePin`, esecuzione smoke test wrapper, registrazione `UpgradeRun`, e tag wrapper che include `upstream_tag + upstream_commit`.

### 1.2A.4 Procedura upgrade upstream (submodule) — checklist + smoke test (STORY-00.1/00.3)

**Obiettivo**
- Rendere ogni bump upstream ripetibile, verificabile, e tracciato (prima/dopo) nel wrapper.

**Prerequisiti**
- Working tree pulito (wrapper) e baseline taggata.
- Target upstream definito come `tag + commit` (non “branch tip”).

**Checklist operativa (wrapper)**
- 1) Pianificazione:
  - Crea un `UpgradeRun` con `status=PLANNED` e campi `upstreamFrom`/`upstreamTo`.
  - Registra un AuditEvent di tipo `STATE_CHANGE` o equivalente (upgrade pianificato).
- 2) Esecuzione bump:
  - Aggiorna il puntamento del submodule `upstream/openclaw` al target `tag+commit`.
  - Aggiorna `UpstreamSourcePin` (repoUrl/tag/commit/pinnedAt) e genera AuditEvent correlato.
- 3) Smoke test upstream (minimo):
  - `openclaw status --all` (diagnostica generale).
  - `openclaw health --json` (snapshot health da gateway, se in esecuzione).
  - `openclaw secrets audit --check` (nessun plaintext/unresolved).
  - `openclaw secrets reload` (swap atomico snapshot runtime, se applicabile).
  - `openclaw memory status --deep` (se la memory plugin è attiva e configurata).
  - `openclaw logs --limit 200` (assenza errori bloccanti, output redatto).
- 4) Smoke test wrapper (contrattuale):
  - Verifica che ogni invocazione upstream nel flusso wrapper erediti `runId` e generi `AuditEvent` correlato (`tenantId/ticketId/runId`).
  - Verifica che il guardrail “Pending Review” sia non bypassabile (errore `POLICY_BLOCK` + audit).
  - Verifica che l’export bundle del wrapper includa `upstream_tag + upstream_commit` (provenienza release).
- 5) Chiusura:
  - Se OK: imposta `UpgradeRun.status=SUCCESS`, registra AuditEvent finale, e tagga la release wrapper includendo `upstream_tag + upstream_commit`.
  - Se KO: ripristina submodule a `upstreamFrom`, imposta `UpgradeRun.status=ROLLED_BACK`, e allega evidenze (export bundle o log redatti).

## 1.3 Attori, Ruoli, Permessi

- Operatore (Owner)
  - Cosa può fare: creare/clonare nodi, gestire routing/porte, approvare in Jira, avviare/fermare servizi, leggere audit e costi, eseguire offboarding.
  - Cosa non può fare: demandare l’approvazione finale all’agente.
- Agente AI (OpenClaw)
  - Cosa può fare: eseguire task in VM, produrre output e artefatti, aggiornare ticket tramite orchestrazione, generare log/audit.
  - Cosa non può fare: bypassare lo stato Pending Review, scrivere segreti in chiaro, eseguire azioni distruttive non tracciate, saturare risorse (kill-switch).
- Sistemi esterni (Jira, Confluence)
  - Cosa possono fare: fornire backlog e documentazione, ricevere aggiornamenti e audit, fungere da “single source of workflow truth”.

## 1.4 Use Case Principali (JTBD)

1) Provisioning di un nuovo nodo cliente (Golden Image → Client Node)
- Trigger: nuovo cliente o nuovo ambiente isolato.
- Successo: VM clonata, rete isolata (no inter-VM), accesso da host funzionante (SSH + web), stack base pronto.
- Fallimento: conflitti IP/porte, ACL errate, impossibilità SSH o perdita accesso web.

2) Esecuzione di un task tracciato (Jira → n8n → OpenClaw)
- Trigger: ticket passa a “Selected for AI”.
- Successo: task processato, artefatti e log allegati, ticket aggiornato e bloccato su “Pending Review”.
- Fallimento: agent loop, permessi insufficienti, integrazione webhook rotta, ticket aggiornato in modo non conforme.

3) Recupero contesto e memoria (Confluence → cache → Vector RAG)
- Trigger: job schedulato o richiesta di contesto per task.
- Successo: dump Markdown aggiornato localmente, indicizzazione coerente, retrieval veloce.
- Fallimento: rate limit Confluence, dump incompleto, mismatch tra cache e indice.

4) Reversibilità e sicurezza operativa (Snapshot/Rollback + Kill-switch)
- Trigger: migrazioni errate, comportamento anomalo, consumo risorse fuori controllo.
- Successo: snapshot ripristinabile, processi terminati, sistema torna in stato sicuro.
- Fallimento: snapshot corrotti, kill-switch non interviene, perdita dati.

5) Offboarding e cold storage
- Trigger: fine contratto.
- Successo: segreti epurati (GDPR/NDA), dati compressi/archiviati, evidenze audit conservate secondo policy.
- Fallimento: residui di segreti, archivi incompleti, perdita tracciabilità.

## 1.5 Requisiti Funzionali (FR)

FR-1: Clonazione nodo cliente
- Descrizione: creare un Client Node a partire da una Golden Image.
- Attori: Operatore.
- Dati coinvolti: profilo cliente (tenant), parametri rete/porte, chiavi SSH.
- Regole: ogni nodo deve essere isolato dagli altri a livello rete.
- Acceptance criteria: creazione ripetibile; nodo raggiungibile via SSH; UI esposte via reverse proxy senza conflitti.

FR-2: Isolamento rete tramite Isolated NAT + ACL
- Descrizione: bloccare traffico verso IP locali (inter-VM/LAN), consentire egress Internet e ingress da host su porte consentite.
- Attori: Operatore, Nodo.
- Dati coinvolti: ACL rules, subnet, allowlist porte.
- Regole: Host → VM consentito (SSH + web); VM → LAN bloccato; VM → Internet consentito.
- Acceptance criteria: test di reachability conforme; nessun traffico laterale verso altre VM.

FR-3: Orchestrazione Jira ↔ n8n ↔ OpenClaw
- Descrizione: un cambio stato Jira innesca workflow n8n che invia task all’agente e aggiorna il ticket.
- Attori: Operatore, Agente, Jira, n8n.
- Dati coinvolti: ticket, commenti, log, allegati.
- Regole: lo stato finale automatico è sempre “Pending Review”.
- Acceptance criteria: ticket aggiornato con audit; nessuna transizione oltre Pending Review senza Operatore.

FR-4: Audit & logging “Grande Fratello”
- Descrizione: tracciare azioni, comandi, output, decisioni e risultati.
- Attori: Agente, Operatore.
- Dati coinvolti: audit events, run logs, riferimenti ticket.
- Regole: nessun segreto in chiaro nei log; correlazione per tenant e ticket.
- Acceptance criteria: audit consultabile e correlabile end-to-end.

FR-5: Vault / blindaggio segreti
- Descrizione: memorizzare credenziali e token evitando esposizione in chiaro.
- Attori: Operatore, Agente.
- Dati coinvolti: secret items, policy di accesso.
- Regole: redazione automatica nei log; accesso minimo necessario.
- Acceptance criteria: segreti non appaiono in log; accesso tracciato.

FR-6: Reverse proxy “Casello”
- Descrizione: esporre interfacce web evitando conflitti di porta sull’host.
- Attori: Operatore.
- Dati coinvolti: routes, hostnames, ports.
- Regole: mapping coerente per-tenant.
- Acceptance criteria: accesso stabile alle UI; nessun conflitto porte.

FR-7: Snapshot/Rollback DB “Time Machine”
- Descrizione: creare backup istantanei e rollback per ripristino rapido.
- Attori: Operatore, Agente.
- Dati coinvolti: snapshot metadata.
- Regole: ogni operazione distruttiva deve avere rollback disponibile.
- Acceptance criteria: restore eseguibile; tempi di ripristino compatibili con uso operativo.

FR-8: Anti-loop watchdog “Kill-Switch”
- Descrizione: rilevare loop CPU/mem e terminare/disabilitare agent/processi.
- Attori: Sistema, Operatore.
- Dati coinvolti: metriche, eventi.
- Regole: soglie configurabili; azione tracciata.
- Acceptance criteria: loop contenuti; nessun consumo illimitato.

FR-9: Onboarding “Client Bootstrapper”
- Descrizione: automatizzare primo avvio (SSH keys, env, token).
- Attori: Operatore.
- Dati coinvolti: bootstrap profile, env vars.
- Regole: validazione input; niente segreti in chiaro.
- Acceptance criteria: onboarding ripetibile e idempotente.

FR-10: Vector RAG (memoria a lungo termine)
- Descrizione: indicizzare e recuperare contesto locale (ticket risolti, pattern, docs).
- Attori: Agente.
- Dati coinvolti: documents, embeddings, retrieval queries.
- Regole: isolamento per-tenant; refresh controllato.
- Acceptance criteria: retrieval veloce; risultati pertinenti.

FR-11: Air-Lock VPN
- Descrizione: garantire che VPN in VM non rompa SSH da host né reverse proxy.
- Attori: Operatore.
- Dati coinvolti: routing rules, vpn state.
- Regole: SSH e proxy devono restare raggiungibili.
- Acceptance criteria: connessione persistente anche con VPN attiva.

FR-12: FinOps/Osservabilità (token e risorse)
- Descrizione: tracciare costi token e consumo risorse, con limiti.
- Attori: Operatore.
- Dati coinvolti: token usage, budgets, alerts.
- Regole: limiti configurabili; alert su anomalie.
- Acceptance criteria: report consultabili; limiti applicati.

FR-13: Offboarding & cold storage
- Descrizione: epurare segreti e archiviare dati a fine contratto.
- Attori: Operatore.
- Dati coinvolti: archive manifest, retention.
- Regole: GDPR/NDA; verifiche di cancellazione.
- Acceptance criteria: nessun segreto residuo; archive completo e verificabile.

## 1.6 Regole di Business (BR)

BR-1: State machine Jira inviolabile
- Quando: un ticket entra nel flusso AI.
- Se: il ticket è “Selected for AI”.
- Allora: l’agente può processare e aggiornare, ma deve fermarsi su “Pending Review”.
- Eccezioni: nessuna; solo l’Operatore può approvare/chiudere e procedere oltre.

BR-2: Tracciabilità totale
- Quando: l’agente esegue un’azione.
- Se: l’azione produce output o modifica stato.
- Allora: deve esistere un audit event correlato a tenant + ticket + run.
- Eccezioni: azioni bloccate se non auditabili.

BR-3: Segreti mai in chiaro
- Quando: vengono usate credenziali/token.
- Se: un output/log contiene pattern sensibile.
- Allora: redazione o blocco con errore e audit.
- Eccezioni: nessuna.

## 1.7 Requisiti Non Funzionali (NFR)

- Sicurezza & Privacy:
- Isolamento per-tenant; segreti protetti e redatti; offboarding con epurazione verificabile.
- Performance:
- Retrieval e operazioni di contesto locali (cache Confluence + indice) per ridurre latenza.
- Affidabilità:
- Snapshot/rollback; ripartenza controllata; idempotenza onboarding.
- Audit/Log (solo a livello requisiti):
- Correlazione end-to-end; retention 365 giorni (configurabile); esportabilità (bundle per tenant/ticket/run).
- Accessibilità:
- Non applicabile (UI principalmente di terze parti / tooling).

## 1.8 Vincoli e Assunzioni

**Vincoli**
- Host Windows 11 con Hyper-V.
- Isolamento rete: blocco inter-VM e verso LAN; consentito egress Internet e ingress da host (SSH + web).
- Gestione operativa via VS Code Remote - SSH.
- “Zero Placeholder” come requisito di delivery quando si genera materiale operativo/automatizzato.

**Assunzioni**
- Jira e Confluence disponibili e accessibili con service account.
- La VM può eseguire Docker.
- Porte e subnet possono essere allocate senza collisioni (range standard definito in Decisioni).

## 1.9 Edge Case (alto livello)

- VPN cliente altera routing e spezza SSH/proxy (coperto da Air-Lock).
- Rate limit o downtime Confluence/Jira.
- Conflitti di porta sull’host, UI non raggiungibili.
- Agent loop o runaway cost (kill-switch + FinOps).
- Snapshot non consistente / restore fallito.

## 1.10 Domande Aperte

Nessuna (decisioni default definite; cambiare solo con ADR esplicito).

## 1.10A Decisioni Default (vincolanti finché non cambiate)

- Interfaccia AI: Open WebUI.
- Vault/Secrets: HashiCorp Vault (single-node per VM) con accesso minimo e redazione obbligatoria nei log.
- Vector RAG: Postgres + pgvector in locale VM, isolamento per-tenant (namespace logico per tenant).
- Audit log: append-only events correlati (tenantId/ticketId/runId), esportabili; retention 365 giorni (configurabile).
- Porte esposte Host → VM: 22 (SSH) e 8080 (reverse proxy). Nessuna altra porta esposta; i servizi UI interni sono raggiunti solo tramite reverse proxy.
- Subnet standard per tenant: 172.29.0.0/16 riservato; allocazione per-tenant /24 (es. 172.29.10.0/24).
- Naming standard:
  - tenantId: `tnt_<slug>`
  - nodeId: `node_<slug>_<nn>`
  - runId: `run_<yyyymmdd>_<seq>`

## 1.10B Matrice di Tracciabilità (FR ↔ UI ↔ Data ↔ Audit)

| FR | UI primaria | Modelli (Data) | Errori chiave (mock) | Audit atteso |
|---|---|---|---|---|
| FR-1 Clonazione nodo | Provisioning (host) + Reverse Proxy | Tenant, Node, AuditEvent | VALIDATION_ERROR, INTEGRATION_ERROR | Eventi COMMAND/STATE_CHANGE correlati a tenantId + nodeId |
| FR-2 Isolamento rete | Provisioning (host) | Node, AuditEvent | POLICY_BLOCK, INTEGRATION_ERROR | Eventi SECURITY con esito test reachability |
| FR-3 Jira ↔ n8n ↔ agente | Jira Ticket Detail + n8n Execution Detail | JiraTicket, AutomationRun, AuditEvent | INTEGRATION_ERROR, POLICY_BLOCK | Audit end-to-end tenantId + ticketId + runId |
| FR-4 Audit & logging | Audit/Logs Explorer | AuditEvent, AutomationRun | FORBIDDEN | Export bundle per tenant/ticket/run |
| FR-5 Vault/Secrets | Interfaccia AI (Open WebUI) + Audit/Logs | SecretItem, AuditEvent | POLICY_BLOCK, FORBIDDEN | Ogni accesso secret → AuditEvent (redacted=true) |
| FR-6 Reverse proxy | Reverse Proxy Routing | ProxyRoute, Node, AuditEvent | VALIDATION_ERROR, INTEGRATION_ERROR | Cambio route → AuditEvent |
| FR-7 Snapshot/Rollback DB | n8n Execution Detail + Audit/Logs | DbSnapshot, AuditEvent, AutomationRun | SNAPSHOT_REQUIRED, INTEGRATION_ERROR | Creazione/restore snapshot → AuditEvent |
| FR-8 Kill-switch | FinOps/Osservabilità + Audit/Logs | WatchdogEvent, AuditEvent | POLICY_BLOCK | Intervento watchdog → WatchdogEvent + AuditEvent |
| FR-9 Onboarding | Provisioning (host) + Interfaccia AI | Tenant, Node, SecretItem, AuditEvent | VALIDATION_ERROR, POLICY_BLOCK | Bootstrap run con evidenza audit |
| FR-10 Vector RAG | n8n Execution Detail + Interfaccia AI | ConfluencePage, AuditEvent | INTEGRATION_ERROR | Sync e retrieval tracciati in audit |
| FR-11 Air-Lock VPN | Provisioning (host) + Reverse Proxy | Node, AuditEvent | INTEGRATION_ERROR | Evidenza “SSH/proxy ok con VPN” in audit |
| FR-12 FinOps/Osservabilità | FinOps/Osservabilità + n8n Execution Detail | TokenUsage, BudgetLimit, AuditEvent | BUDGET_BREACH | Breach → blocco + AuditEvent correlato |
| FR-13 Offboarding | Audit/Logs + Provisioning (host) | OffboardingJob, Tenant, AuditEvent | INTEGRATION_ERROR | Export audit + purge segreti + Tenant=OFFBOARDED |

## 1.11 Istruzioni per l’AI builder

Leggi questi file nell’ordine:
1. `openclaw/1_PRD_Master.md`
2. `openclaw/2_UI_Architecture_and_Flows.md`
3. `openclaw/3_Data_Models_and_Mock_Data.md`
4. `docs/CONTEXT.md` (governance + “Fase Codice”)

Regole:
- Non inventare requisiti o modelli dati: se manca qualcosa, elenca domande.
- Rispetta scope e non-scope.
- La fase codice non avviene in questo repo: usa i master come contratto read-only e segui `docs/CONTEXT.md` (sezione “Fase Codice”).

## 1.12 Backlog Implementazione (Jira)

EPIC-00 — Upstream OpenClaw + Wrap System (prodotto unico, modulo separato e aggiornabile)
- Scope: 1.2A (strategia upstream/wrapper) + vincoli di governance
- Stories:
  - STORY-00.1 Pin upstream (submodule) + registrazione versione/commit + procedura update
    - Done: `UpstreamSourcePin` aggiornato (tag+commit+pinnedAt) e AuditEvent correlato
    - Done: `UpgradeRun` creato e chiuso (SUCCESS/FAILED/ROLLED_BACK) con evidenze
  - STORY-00.2 Definizione boundary: adapter/contract tra wrapper e upstream (API/CLI/config/plugin)
  - STORY-00.3 Compatibilità upgrade: checklist e test smoke per ogni bump upstream
    - Done: eseguita la checklist di sezione 1.2A.4
    - Done: smoke test upstream minimi ok + smoke test wrapper contrattuali ok
  - STORY-00.4 Tracciabilità release: wrapper tagga sempre “upstream_tag + upstream_commit”

### 1.12A Gap analysis (FR ↔ upstream OpenClaw)

Legenda copertura upstream:
- Coperto: esiste una capability upstream direttamente riusabile dal wrapper (senza fork).
- Parziale: esiste una capability upstream, ma manca contratto/tenancy/correlazione/policy specifica del wrapper.
- Mancante: non esiste capability upstream utile; da implementare nel wrapper.

| FR | Copertura upstream | Evidenza upstream (non esaustiva) | Gap / lavoro wrapper (unità mancanti) |
|---|---|---|---|
| FR-1 Clonazione nodo cliente | Mancante | N/A | Provisioning Hyper‑V (Golden Image → Client Node), naming/allocazione, audit provisioning |
| FR-2 Isolamento rete NAT+ACL | Mancante | N/A | Isolated NAT + ACL policy (no LAN/no inter‑VM), test reachability, audit SECURITY |
| FR-3 Jira ↔ n8n ↔ agente | Mancante | N/A | Webhook Jira + pipeline n8n + guardrail Pending Review + correlazione end‑to‑end |
| FR-4 Audit & logging | Parziale | `docs/tools/trajectory.md`, `docs/cli/logs.md` | AuditEvent business (tenant/ticket/run), redazione, Explorer, export bundle “contrattuale” per tenant/ticket/run |
| FR-5 Vault/Secrets | Parziale | `docs/cli/secrets.md` | Integrazione Vault (decisione default), policy accesso minimo, audit accessi secret nel modello wrapper |
| FR-6 Reverse proxy “Casello” | Mancante | N/A | Reverse proxy unico ingress 8080, registry route per-tenant, health, audit cambi |
| FR-7 Snapshot/Rollback DB | Mancante | N/A | DbSnapshot create/restore + enforcement `SNAPSHOT_REQUIRED` + audit correlato |
| FR-8 Kill-switch / watchdog | Parziale | `/kill` e gestione run/subagent in `docs/tools/subagents.md` | Watchdog CPU/mem (soglie), STOP_RUN/DISABLE_AGENT, WatchdogEvent + AuditEvent |
| FR-9 Onboarding bootstrap | Mancante | N/A | Bootstrap per-tenant (SSH keys/env/token), idempotenza, validazione input, audit bootstrap |
| FR-10 Vector RAG | Parziale | `docs/cli/memory.md` | Confluence sync/cache + pgvector (decisione default), isolamento per-tenant, audit sync/retrieval |
| FR-11 Air-Lock VPN | Mancante | N/A | Routing rules per VPN in VM senza rompere SSH/proxy, test automatici, audit esito |
| FR-12 FinOps/Osservabilità | Parziale | `/usage` in `docs/tools/slash-commands.md` | Tracking per tenant/ticket/run + BudgetLimit enforcement (`BUDGET_BREACH` → STOP_RUNS) + audit |
| FR-13 Offboarding & cold storage | Parziale | `docs/cli/backup.md`, `docs/cli/secrets.md` | Offboarding per tenant (purge segreti + export audit bundle + archive manifest), stato Tenant=OFFBOARDED |

EPIC-01 — Provisioning Golden Image & Client Node (Hyper-V + accesso host)
- Scope: FR-1, FR-2, FR-9, FR-11
- Stories:
  - STORY-01.1 Clonazione nodo cliente (Golden Image → Client Node)
  - STORY-01.2 Isolamento rete Isolated NAT + ACL (no LAN / no inter-VM)
  - STORY-01.3 Onboarding bootstrap (SSH keys + env + token)
  - STORY-01.4 Air-Lock VPN (VPN in VM senza rompere SSH/proxy)

EPIC-02 — Orchestrazione Jira ↔ n8n ↔ OpenClaw (state machine inviolabile)
- Scope: FR-3 (vincoli BR-1/BR-2)
- Stories:
  - STORY-02.1 Webhook Jira: Selected for AI → n8n
  - STORY-02.2 n8n pipeline (ingest → RAG → execute → update Jira)
  - STORY-02.3 Guardrail Pending Review (blocco hard + POLICY_BLOCK)

EPIC-03 — Audit & Logging + Export bundle
- Scope: FR-4
- Stories:
  - STORY-03.1 AuditEvent append-only + correlazione obbligatoria
  - STORY-03.2 Audit Explorer + export bundle per tenant/ticket/run

EPIC-04 — Vault/Secrets (no leak)
- Scope: FR-5
- Stories:
  - STORY-04.1 SecretItem contract + access audit + redazione obbligatoria

EPIC-05 — Reverse Proxy “Casello” (single ingress 8080)
- Scope: FR-6
- Stories:
  - STORY-05.1 ProxyRoute registry + health + audit modifiche route

EPIC-06 — Time Machine DB (snapshot/rollback)
- Scope: FR-7
- Stories:
  - STORY-06.1 DbSnapshot create/restore + policy SNAPSHOT_REQUIRED

EPIC-07 — Kill-Switch / Watchdog
- Scope: FR-8
- Stories:
  - STORY-07.1 WatchdogEvent + azioni STOP_RUN/DISABLE_AGENT + AuditEvent correlato

EPIC-08 — Vector RAG (Confluence cache → pgvector)
- Scope: FR-10
- Stories:
  - STORY-08.1 Confluence sync/cache + ingestion + audit

EPIC-09 — FinOps/Osservabilità (token + budget)
- Scope: FR-12
- Stories:
  - STORY-09.1 TokenUsage tracking per run/ticket/tenant
  - STORY-09.2 BudgetLimit enforcement (BUDGET_BREACH → STOP_RUNS + audit)

EPIC-10 — Offboarding & Cold Storage
- Scope: FR-13
- Stories:
  - STORY-10.1 OffboardingJob (purge → export audit bundle → archive) + Tenant OFFBOARDED
