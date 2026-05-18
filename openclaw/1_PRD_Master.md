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

## 1.11 Istruzioni per l’AI builder

Leggi questi file nell’ordine:
1. `openclaw/1_PRD_Master.md`
2. `openclaw/2_UI_Architecture_and_Flows.md`
3. `openclaw/3_Data_Models_and_Mock_Data.md`

Regole:
- Non inventare requisiti o modelli dati: se manca qualcosa, elenca domande.
- Rispetta scope e non-scope.
