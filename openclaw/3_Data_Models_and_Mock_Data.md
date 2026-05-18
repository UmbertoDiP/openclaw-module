---
title: Data Models & Mock Data — OpenClaw
audience: AI Builder
version: 0.1
---

# 3) Data Models & Mock Data — OpenClaw

## 3.1 Convenzioni

- Ogni modello: nome, significato business, campi, vincoli, mock JSON singolo + lista.
- Includere payload errore (validazione/permessi) per guidare stati UI.

## 3.2 Modelli Dati

### Model: Tenant

**Descrizione business**
- Un cliente/contesto isolato (“Client Node” dedicato). Base di correlazione per audit, segreti, run e costi.

**Schema**
- id: string (required) es. `tnt_cliente_x`
- name: string (required)
- status: string (required) enum: `ACTIVE|SUSPENDED|OFFBOARDED`
- createdAt: string (ISO-8601) (required)
- updatedAt: string (ISO-8601) (required)

**Vincoli**
- id unico; isolamento logico per tenant su ogni risorsa.

**Mock JSON (singolo)**
```json
{
  "id": "tnt_cliente_x",
  "name": "Cliente X",
  "status": "ACTIVE",
  "createdAt": "2026-05-18T21:00:00Z",
  "updatedAt": "2026-05-18T21:00:00Z"
}
```

**Mock JSON (lista)**
```json
[
  {
    "id": "tnt_cliente_x",
    "name": "Cliente X",
    "status": "ACTIVE",
    "createdAt": "2026-05-18T21:00:00Z",
    "updatedAt": "2026-05-18T21:00:00Z"
  }
]
```

### Model: Node

**Descrizione business**
- Una VM per-tenant (Hyper-V) clonata dalla Golden Image, con isolamento rete e stack servizi.

**Schema**
- id: string (required) es. `node_cliente_x_01`
- tenantId: string (required)
- mode: string (required) enum: `GOLDEN_IMAGE|CLIENT_NODE`
- status: string (required) enum: `PROVISIONING|READY|ERROR|OFFBOARDED`
- hostAccess:
  - sshEnabled: boolean (required)
  - webBaseUrl: string (required)
- networkPolicy:
  - egressInternet: boolean (required)
  - blockLocalRanges: boolean (required)
  - allowedIngressPortsFromHost: number[] (required)
- createdAt: string (ISO-8601) (required)
- updatedAt: string (ISO-8601) (required)

**Vincoli**
- `tenantId` deve esistere; le porte ingress devono essere una allowlist esplicita.

**Mock JSON (singolo)**
```json
{
  "id": "node_cliente_x_01",
  "tenantId": "tnt_cliente_x",
  "mode": "CLIENT_NODE",
  "status": "READY",
  "hostAccess": {
    "sshEnabled": true,
    "webBaseUrl": "http://localhost:8080/"
  },
  "networkPolicy": {
    "egressInternet": true,
    "blockLocalRanges": true,
    "allowedIngressPortsFromHost": [22, 8080]
  },
  "createdAt": "2026-05-18T21:10:00Z",
  "updatedAt": "2026-05-18T21:12:00Z"
}
```

**Mock JSON (lista)**
```json
[
  {
    "id": "node_cliente_x_01",
    "tenantId": "tnt_cliente_x",
    "mode": "CLIENT_NODE",
    "status": "READY",
    "hostAccess": {
      "sshEnabled": true,
      "webBaseUrl": "http://localhost:8080/"
    },
    "networkPolicy": {
      "egressInternet": true,
      "blockLocalRanges": true,
      "allowedIngressPortsFromHost": [22, 8080]
    },
    "createdAt": "2026-05-18T21:10:00Z",
    "updatedAt": "2026-05-18T21:12:00Z"
  }
]
```

### Model: JiraTicket

**Descrizione business**
- Unità di lavoro governata dalla state machine (Backlog → Selected for AI → Processing → Pending Review).

**Schema**
- id: string (required) es. `JIRA-123`
- tenantId: string (required)
- status: string (required) enum: `BACKLOG|SELECTED_FOR_AI|PROCESSING|PENDING_REVIEW|DONE`
- summary: string (required)
- description: string (optional)
- approvalRequired: boolean (required)
- aiProcessed: boolean (required)
- createdAt: string (ISO-8601) (required)
- updatedAt: string (ISO-8601) (required)

**Vincoli**
- transizioni oltre `PENDING_REVIEW` solo con azione Operatore.

**Mock JSON (singolo)**
```json
{
  "id": "JIRA-123",
  "tenantId": "tnt_cliente_x",
  "status": "PENDING_REVIEW",
  "summary": "Hardening rete + verifica accesso SSH",
  "description": "Applicare policy Isolated NAT e assicurare accesso host→VM su 22/8080.",
  "approvalRequired": true,
  "aiProcessed": true,
  "createdAt": "2026-05-18T21:15:00Z",
  "updatedAt": "2026-05-18T21:40:00Z"
}
```

**Mock JSON (lista)**
```json
[
  {
    "id": "JIRA-123",
    "tenantId": "tnt_cliente_x",
    "status": "PENDING_REVIEW",
    "summary": "Hardening rete + verifica accesso SSH",
    "description": "Applicare policy Isolated NAT e assicurare accesso host→VM su 22/8080.",
    "approvalRequired": true,
    "aiProcessed": true,
    "createdAt": "2026-05-18T21:15:00Z",
    "updatedAt": "2026-05-18T21:40:00Z"
  }
]
```

### Model: AutomationRun

**Descrizione business**
- Esecuzione end-to-end orchestrata (n8n) correlata a un ticket: ingest contesto → esecuzione agente → update Jira.

**Schema**
- id: string (required) es. `run_20260518_001`
- tenantId: string (required)
- ticketId: string (required)
- status: string (required) enum: `RUNNING|SUCCEEDED|FAILED|ABORTED`
- startedAt: string (ISO-8601) (required)
- finishedAt: string (ISO-8601) (optional)
- steps: { name: string, status: string, startedAt: string, finishedAt: string, errorCode?: string }[] (required)

**Vincoli**
- ogni step deve essere auditabile; payload sensibili redatti.

**Mock JSON (singolo)**
```json
{
  "id": "run_20260518_001",
  "tenantId": "tnt_cliente_x",
  "ticketId": "JIRA-123",
  "status": "SUCCEEDED",
  "startedAt": "2026-05-18T21:20:00Z",
  "finishedAt": "2026-05-18T21:35:00Z",
  "steps": [
    { "name": "jira_webhook_ingest", "status": "SUCCEEDED", "startedAt": "2026-05-18T21:20:00Z", "finishedAt": "2026-05-18T21:20:05Z" },
    { "name": "confluence_sync_or_cache", "status": "SUCCEEDED", "startedAt": "2026-05-18T21:20:05Z", "finishedAt": "2026-05-18T21:21:00Z" },
    { "name": "rag_retrieval", "status": "SUCCEEDED", "startedAt": "2026-05-18T21:21:00Z", "finishedAt": "2026-05-18T21:21:03Z" },
    { "name": "agent_execute", "status": "SUCCEEDED", "startedAt": "2026-05-18T21:21:03Z", "finishedAt": "2026-05-18T21:33:00Z" },
    { "name": "jira_update_pending_review", "status": "SUCCEEDED", "startedAt": "2026-05-18T21:33:00Z", "finishedAt": "2026-05-18T21:35:00Z" }
  ]
}
```

**Mock JSON (lista)**
```json
[
  {
    "id": "run_20260518_001",
    "tenantId": "tnt_cliente_x",
    "ticketId": "JIRA-123",
    "status": "SUCCEEDED",
    "startedAt": "2026-05-18T21:20:00Z",
    "finishedAt": "2026-05-18T21:35:00Z",
    "steps": [
      { "name": "jira_webhook_ingest", "status": "SUCCEEDED", "startedAt": "2026-05-18T21:20:00Z", "finishedAt": "2026-05-18T21:20:05Z" }
    ]
  }
]
```

### Model: AuditEvent

**Descrizione business**
- Evento atomico di audit correlato a tenant + ticket + run: comandi, output, decisioni e blocchi policy.

**Schema**
- id: string (required) es. `aud_0001`
- tenantId: string (required)
- ticketId: string (optional)
- runId: string (optional)
- type: string (required) enum: `COMMAND|OUTPUT|STATE_CHANGE|POLICY_BLOCK|ERROR|SECURITY`
- message: string (required)
- redacted: boolean (required)
- createdAt: string (ISO-8601) (required)

**Vincoli**
- nessun segreto in chiaro; se redacted=false e contiene segreti → invalido.

**Mock JSON (singolo)**
```json
{
  "id": "aud_0001",
  "tenantId": "tnt_cliente_x",
  "ticketId": "JIRA-123",
  "runId": "run_20260518_001",
  "type": "STATE_CHANGE",
  "message": "Ticket transizionato a PENDING_REVIEW",
  "redacted": true,
  "createdAt": "2026-05-18T21:35:00Z"
}
```

**Mock JSON (lista)**
```json
[
  {
    "id": "aud_0001",
    "tenantId": "tnt_cliente_x",
    "ticketId": "JIRA-123",
    "runId": "run_20260518_001",
    "type": "STATE_CHANGE",
    "message": "Ticket transizionato a PENDING_REVIEW",
    "redacted": true,
    "createdAt": "2026-05-18T21:35:00Z"
  }
]
```

### Model: SecretItem

**Descrizione business**
- Credenziale/token per-tenant. Deve essere accessibile in modo controllato e mai comparire in chiaro nei log.

**Schema**
- id: string (required) es. `sec_jira_api_token`
- tenantId: string (required)
- kind: string (required) enum: `API_TOKEN|PASSWORD|SSH_KEY|CERT`
- label: string (required)
- valueRef: string (required) riferimento al vault (non il valore)
- createdAt: string (ISO-8601) (required)
- updatedAt: string (ISO-8601) (required)

**Vincoli**
- `valueRef` non deve contenere il valore; accesso sempre auditato.

**Mock JSON (singolo)**
```json
{
  "id": "sec_jira_api_token",
  "tenantId": "tnt_cliente_x",
  "kind": "API_TOKEN",
  "label": "Jira Service Account Token",
  "valueRef": "vault://tnt_cliente_x/jira/service_token",
  "createdAt": "2026-05-18T21:05:00Z",
  "updatedAt": "2026-05-18T21:05:00Z"
}
```

**Mock JSON (lista)**
```json
[
  {
    "id": "sec_jira_api_token",
    "tenantId": "tnt_cliente_x",
    "kind": "API_TOKEN",
    "label": "Jira Service Account Token",
    "valueRef": "vault://tnt_cliente_x/jira/service_token",
    "createdAt": "2026-05-18T21:05:00Z",
    "updatedAt": "2026-05-18T21:05:00Z"
  }
]
```

### Model: ConfluencePage

**Descrizione business**
- Pagina Confluence sincronizzata in cache Markdown locale per RAG.

**Schema**
- id: string (required) es. `conf_8891`
- tenantId: string (required)
- title: string (required)
- sourceUrl: string (required)
- markdownPath: string (required)
- syncedAt: string (ISO-8601) (required)
- createdAt: string (ISO-8601) (required)
- updatedAt: string (ISO-8601) (required)

**Vincoli**
- `markdownPath` deve puntare a cache locale; refresh schedulato.

**Mock JSON (singolo)**
```json
{
  "id": "conf_8891",
  "tenantId": "tnt_cliente_x",
  "title": "ADR-001: Isolated NAT + ACL",
  "sourceUrl": "https://confluence.example/wiki/spaces/ARCH/pages/8891",
  "markdownPath": "/docs_cache/confluence/ADR-001.md",
  "syncedAt": "2026-05-18T21:25:00Z",
  "createdAt": "2026-05-18T21:25:00Z",
  "updatedAt": "2026-05-18T21:25:00Z"
}
```

**Mock JSON (lista)**
```json
[
  {
    "id": "conf_8891",
    "tenantId": "tnt_cliente_x",
    "title": "ADR-001: Isolated NAT + ACL",
    "sourceUrl": "https://confluence.example/wiki/spaces/ARCH/pages/8891",
    "markdownPath": "/docs_cache/confluence/ADR-001.md",
    "syncedAt": "2026-05-18T21:25:00Z",
    "createdAt": "2026-05-18T21:25:00Z",
    "updatedAt": "2026-05-18T21:25:00Z"
  }
]
```

## 3.3 Error Payload (mock)

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Campo non valido",
    "details": [
      { "field": "name", "reason": "REQUIRED" }
    ]
  }
}
```
