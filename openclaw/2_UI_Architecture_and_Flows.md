---
title: UI Architecture & Flows — OpenClaw
audience: AI Builder
version: 0.1
---

# 2) UI Architecture & Flows — OpenClaw

## 2.1 Mappa Viste (IA)

- Jira (Board + Ticket detail)
- n8n (Workflow list + Execution detail)
- Interfaccia AI (DA DECIDERE: Open WebUI o AnythingLLM)
- Reverse proxy (routing verso UI)
- Audit/Logs (vista consultazione: DA DECIDERE se UI dedicata o via strumenti esistenti)
- FinOps/Osservabilità (dashboard: DA DECIDERE)
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
- Vista eventi audit filtrabile per tenant/ticket/run (DA DECIDERE)
- Vista costi/token e anomalie (DA DECIDERE)

## 2.4 Stati UI (obbligatori per ogni vista)

- Loading: cosa mostra e cosa è disabilitato
- Empty: messaggio + CTA
- Success: contenuto + azioni
- Error: messaggio + retry + fallback

## 2.5 Specifica per Vista (template)

### Vista: Jira — Ticket Detail (Selected for AI / Processing / Pending Review)

- Scopo:
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
- Empty: nessun ticket associato (DA DECIDERE comportamento).
- Success: sessione attiva con strumenti disponibili.
- Error: perdita connessione/permessi; fallback su logs/audit.

## 2.6 Regole UI (derivate da business)

- Abilitazioni/disabilitazioni azioni: oltre “Pending Review” solo Operatore; rerun n8n solo per ticket validi.
- Visibilità per ruolo: separazione per tenant (obbligatoria); audit sempre consultabile da Operatore.
- Stati read-only: log e payload devono essere redatti; segreti mai mostrati.
