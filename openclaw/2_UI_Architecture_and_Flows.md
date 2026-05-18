---
title: UI Architecture & Flows — OpenClaw
audience: AI Builder
version: 0.1
---

# 2) UI Architecture & Flows — OpenClaw

## 2.1 Mappa Viste (IA)

- Jira (Board + Ticket detail)
- n8n (Workflow list + Execution detail)
- Interfaccia AI (Open WebUI)
- Reverse proxy (routing verso UI)
- Audit/Logs (explore + ricerca, per tenant/ticket/run)
- FinOps/Osservabilità (dashboard costi/token + risorse)
- Provisioning (azione da host: script/command surface, non UI web)

## 2.2 Navigazione (happy path)

1) Jira: Ticket passa a “Selected for AI” → webhook
2) n8n: Workflow riceve evento → prepara contesto (Confluence cache/RAG) → invia comando a OpenClaw
3) OpenClaw: esegue → produce log/artefatti → risponde a n8n
4) n8n: aggiorna ticket Jira con commento/audit → transizione a “Pending Review”
5) Operatore: apre ticket “Pending Review” → valida output → approva (manuale) o richiede iterazione

## 2.3 Componenti UI (macro)

- Board e card ticket (Jira)
- Pannello esecuzioni e log step-by-step (n8n)
- Chat/sessioni e strumenti operativi (Interfaccia AI)
- Routing/ingress per UI (reverse proxy)
- Vista eventi audit filtrabile per tenant/ticket/run
- Vista costi/token e anomalie

## 2.4 Stati UI (obbligatori per ogni vista)

- Loading: cosa mostra e cosa è disabilitato
- Empty: messaggio + CTA
- Success: contenuto + azioni
- Error: messaggio + retry + fallback

## 2.5 Specifica per Vista (template)

## 2.5A Matrice Vista ↔ Data ↔ Errori ↔ Audit

| Vista | Modelli (Data) | Errori chiave (mock) | Audit atteso |
|---|---|---|---|
| Jira — Ticket Detail | JiraTicket, AuditEvent | INTEGRATION_ERROR, POLICY_BLOCK, FORBIDDEN | Eventi STATE_CHANGE e correlazione ticketId/runId |
| n8n — Execution Detail | AutomationRun, JiraTicket, AuditEvent, DbSnapshot, TokenUsage | INTEGRATION_ERROR, SNAPSHOT_REQUIRED, BUDGET_BREACH | Timeline step con runId; export log redatti |
| Interfaccia AI — Sessione Operativa | Tenant, JiraTicket, SecretItem, AuditEvent | POLICY_BLOCK, FORBIDDEN, AUTH_REQUIRED | Ogni azione/tool e accesso secret → AuditEvent redacted |
| Reverse Proxy — Routing | ProxyRoute, Node, AuditEvent | VALIDATION_ERROR, INTEGRATION_ERROR | Modifica route e health check → AuditEvent |
| Audit/Logs — Explorer | AuditEvent, AutomationRun, JiraTicket | FORBIDDEN | Export bundle per tenant/ticket/run |
| FinOps/Osservabilità — Dashboard | TokenUsage, BudgetLimit, WatchdogEvent, AuditEvent | BUDGET_BREACH, INTEGRATION_ERROR | Modifiche budget/azioni contenimento → AuditEvent |
### Vista: Jira — Ticket Detail (Selected for AI / Processing / Pending Review)


- Scopo: rappresentare la singola unità di lavoro tracciata e governare lo stato macchina.
- Chi la usa: Operatore, (in lettura) Agente tramite integrazione.
- Dati richiesti: ticket, stato, campi AI_Processed/Approval_Required (se presenti), allegati, commenti, link a audit run.
- Azioni principali: spostare a Selected for AI; revisionare output in Pending Review; approvare/chiudere o richiedere modifiche.
- Validazioni: transizioni consentite; blocco oltre Pending Review senza azione Operatore.
- Messaggi (success/error): esito transizione; errori webhook/integrations; alert su policy violata.

**Stati UI**
- Loading: dettaglio ticket in caricamento; azioni disabilitate.
- Empty: ticket non trovato o permessi insufficienti.
- Success: dettaglio completo + CTA coerenti con stato.
- Error: errore integrazione o permessi; retry o escalation.

### Vista: n8n — Execution Detail (run per ticket)

- Scopo: vedere pipeline step-by-step (ingest, RAG, esecuzione agente, update Jira).
- Chi la usa: Operatore.
- Dati richiesti: execution id, step logs, input/output payload (redatto), link a ticket/tenant.
- Azioni principali: rerun controllato; stop/abort; export log.
- Validazioni: nessun secret in chiaro; rerun solo su ticket consentiti.
- Messaggi (success/error): run completata; run fallita con causa; policy violation.

**Stati UI**
- Loading: execution in caricamento; stop disponibile se running.
- Empty: nessuna esecuzione per ticket.
- Success: timeline step con payload redatti + artefatti.
- Error: step error; CTA retry/rollback.

### Vista: Interfaccia AI — Sessione Operativa

- Scopo: interazione controllata con l’agente e consultazione output.
- Chi la usa: Operatore (primario), Agente (via tool).
- Dati richiesti: contesto per-tenant, ticket corrente, output tool, log redatti.
- Azioni principali: inviare istruzioni; consultare run; generare output da allegare al ticket.
- Validazioni: policy di non-esposizione segreti; blocco azioni distruttive senza snapshot/audit.
- Messaggi (success/error): policy block; tool failure; contesto mancante.

**Stati UI**
- Loading: contesto in setup; input disabilitato se non pronto.
- Empty: nessun ticket associato → mostra selettore tenant/ticket o istruzione di associare un ticket.
- Success: sessione attiva con strumenti disponibili.
- Error: perdita connessione/permessi; fallback su logs/audit.

### Vista: Reverse Proxy — Routing verso UI

- Scopo: esporre tutte le UI interne tramite un singolo punto di ingresso (porta 8080) evitando conflitti.
- Chi la usa: Operatore.
- Dati richiesti: elenco servizi, route attive, health check per servizio, mapping host/path.
- Azioni principali: abilitare/disabilitare route; verificare health; consultare errori di routing.
- Validazioni: nessuna esposizione diretta di porte interne; route isolate per tenant.
- Messaggi (success/error): servizio down; route non valida; conflitto mapping.

**Stati UI**
- Loading: configurazione in caricamento; azioni disabilitate.
- Empty: nessuna route configurata.
- Success: routes attive con stato health.
- Error: errore di proxy/routing con suggerimento di remediation.

### Vista: Audit/Logs — Explorer (tenant/ticket/run)

- Scopo: consultare eventi audit e log di esecuzione correlati, con ricerca e filtri.
- Chi la usa: Operatore.
- Dati richiesti: audit events, riferimenti ticket/run, log redatti, export bundle.
- Azioni principali: filtrare per tenant/ticket/run; aprire dettaglio evento; esportare bundle; verificare redazione.
- Validazioni: log sempre redatti; accesso solo Operatore; isolamento per tenant.
- Messaggi (success/error): export fallito; permessi; log mancanti.

**Stati UI**
- Loading: eventi in caricamento; export disabilitato.
- Empty: nessun evento per i filtri.
- Success: timeline + dettaglio evento.
- Error: storage non raggiungibile; retry.

### Vista: FinOps/Osservabilità — Dashboard

- Scopo: vedere consumo risorse e token, budget e anomalie; supportare kill-switch decisionale.
- Chi la usa: Operatore.
- Dati richiesti: token usage per run/ticket/tenant, budget limit, metriche CPU/RAM/Disk, alerts.
- Azioni principali: impostare budget; riconoscere anomalie; attivare azioni di contenimento (es. stop run).
- Validazioni: threshold configurabili; audit delle modifiche a budget/limit.
- Messaggi (success/error): soglia superata; metriche non disponibili; aggiornamento budget fallito.

**Stati UI**
- Loading: metriche in caricamento; azioni limitate.
- Empty: nessun dato (nuovo tenant) → CTA “inizializza baseline”.
- Success: dashboard con trend e alert.
- Error: collector down; fallback su audit.

## 2.6 Regole UI (derivate da business)

- Abilitazioni/disabilitazioni azioni: oltre “Pending Review” solo Operatore; rerun n8n solo per ticket validi.
- Visibilità per ruolo: separazione per tenant (obbligatoria); audit sempre consultabile da Operatore.
- Stati read-only: log e payload devono essere redatti; segreti mai mostrati.
