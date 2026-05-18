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

### Model: ProxyRoute

**Descrizione business**
- Regola di routing del reverse proxy (porta 8080) verso un servizio interno, senza esporre porte aggiuntive verso l’host.

**Schema**
- id: string (required) es. `route_openwebui`
- tenantId: string (required)
- entrypoint: string (required) enum: `HOST_8080`
- match: { host?: string, pathPrefix?: string } (required)
- target: { service: string, internalUrl: string } (required)
- enabled: boolean (required)
- createdAt: string (ISO-8601) (required)
- updatedAt: string (ISO-8601) (required)

**Vincoli**
- `internalUrl` non deve essere raggiungibile direttamente dall’host; tutte le UI passano dal proxy.

**Mock JSON (singolo)**
```json
{
  "id": "route_openwebui",
  "tenantId": "tnt_cliente_x",
  "entrypoint": "HOST_8080",
  "match": { "pathPrefix": "/ai" },
  "target": { "service": "openwebui", "internalUrl": "http://openwebui:3000" },
  "enabled": true,
  "createdAt": "2026-05-18T21:12:00Z",
  "updatedAt": "2026-05-18T21:12:00Z"
}
```

**Mock JSON (lista)**
```json
[
  {
    "id": "route_openwebui",
    "tenantId": "tnt_cliente_x",
    "entrypoint": "HOST_8080",
    "match": { "pathPrefix": "/ai" },
    "target": { "service": "openwebui", "internalUrl": "http://openwebui:3000" },
    "enabled": true,
    "createdAt": "2026-05-18T21:12:00Z",
    "updatedAt": "2026-05-18T21:12:00Z"
  },
  {
    "id": "route_n8n",
    "tenantId": "tnt_cliente_x",
    "entrypoint": "HOST_8080",
    "match": { "pathPrefix": "/n8n" },
    "target": { "service": "n8n", "internalUrl": "http://n8n:5678" },
    "enabled": true,
    "createdAt": "2026-05-18T21:12:00Z",
    "updatedAt": "2026-05-18T21:12:00Z"
  }
]
```

### Model: DbSnapshot

**Descrizione business**
- Snapshot coerente (Time Machine) per ripristino rapido in caso di migrazioni errate o test distruttivi.

**Schema**
- id: string (required) es. `snap_20260518_001`
- tenantId: string (required)
- scope: string (required) enum: `ALL|POSTGRES|REDIS|OTHER`
- reason: string (required)
- createdBy: string (required) enum: `OPERATOR|AGENT|AUTOMATION`
- status: string (required) enum: `CREATED|FAILED|DELETED|RESTORED`
- createdAt: string (ISO-8601) (required)
- updatedAt: string (ISO-8601) (required)

**Vincoli**
- Prima di azioni potenzialmente distruttive deve esistere uno snapshot valido o una motivazione auditata.

**Mock JSON (singolo)**
```json
{
  "id": "snap_20260518_001",
  "tenantId": "tnt_cliente_x",
  "scope": "POSTGRES",
  "reason": "Prima di applicare migrazione automatica",
  "createdBy": "AUTOMATION",
  "status": "CREATED",
  "createdAt": "2026-05-18T21:22:00Z",
  "updatedAt": "2026-05-18T21:22:00Z"
}
```

**Mock JSON (lista)**
```json
[
  {
    "id": "snap_20260518_001",
    "tenantId": "tnt_cliente_x",
    "scope": "POSTGRES",
    "reason": "Prima di applicare migrazione automatica",
    "createdBy": "AUTOMATION",
    "status": "CREATED",
    "createdAt": "2026-05-18T21:22:00Z",
    "updatedAt": "2026-05-18T21:22:00Z"
  },
  {
    "id": "snap_20260518_002",
    "tenantId": "tnt_cliente_x",
    "scope": "ALL",
    "reason": "Prima di rerun workflow su ticket critico",
    "createdBy": "OPERATOR",
    "status": "CREATED",
    "createdAt": "2026-05-18T21:50:00Z",
    "updatedAt": "2026-05-18T21:50:00Z"
  }
]
```

### Model: WatchdogEvent

**Descrizione business**
- Evento generato dal kill-switch quando rileva loop/anomalia e interviene (stop processo/run).

**Schema**
- id: string (required) es. `wd_0001`
- tenantId: string (required)
- runId: string (optional)
- severity: string (required) enum: `INFO|WARN|CRITICAL`
- signal: string (required) enum: `CPU_SPIKE|MEM_SPIKE|DISK_FULL|RUNAWAY_CALLS|HANG`
- action: string (required) enum: `ALERT_ONLY|STOP_RUN|DISABLE_AGENT|RESTART_SERVICE`
- createdAt: string (ISO-8601) (required)

**Vincoli**
- Ogni azione del watchdog deve generare anche un AuditEvent correlato.

**Mock JSON (singolo)**
```json
{
  "id": "wd_0001",
  "tenantId": "tnt_cliente_x",
  "runId": "run_20260518_001",
  "severity": "CRITICAL",
  "signal": "MEM_SPIKE",
  "action": "STOP_RUN",
  "createdAt": "2026-05-18T21:28:00Z"
}
```

**Mock JSON (lista)**
```json
[
  {
    "id": "wd_0001",
    "tenantId": "tnt_cliente_x",
    "runId": "run_20260518_001",
    "severity": "CRITICAL",
    "signal": "MEM_SPIKE",
    "action": "STOP_RUN",
    "createdAt": "2026-05-18T21:28:00Z"
  },
  {
    "id": "wd_0002",
    "tenantId": "tnt_cliente_x",
    "runId": "run_20260518_001",
    "severity": "WARN",
    "signal": "CPU_SPIKE",
    "action": "ALERT_ONLY",
    "createdAt": "2026-05-18T21:27:30Z"
  }
]
```

### Model: TokenUsage

**Descrizione business**
- Consumo token e costo stimato, tracciato per tenant e correlabile a run/ticket.

**Schema**
- id: string (required) es. `tok_20260518_001`
- tenantId: string (required)
- runId: string (optional)
- ticketId: string (optional)
- provider: string (required) enum: `OPENAI|GEMINI|ANTHROPIC|OTHER`
- model: string (required)
- inputTokens: number (required)
- outputTokens: number (required)
- totalTokens: number (required)
- estimatedCostUsd: number (required)
- createdAt: string (ISO-8601) (required)

**Vincoli**
- Il consumo deve essere aggregabile per budget e alert.

**Mock JSON (singolo)**
```json
{
  "id": "tok_20260518_001",
  "tenantId": "tnt_cliente_x",
  "runId": "run_20260518_001",
  "ticketId": "JIRA-123",
  "provider": "OPENAI",
  "model": "gpt-5",
  "inputTokens": 12000,
  "outputTokens": 4500,
  "totalTokens": 16500,
  "estimatedCostUsd": 1.85,
  "createdAt": "2026-05-18T21:35:00Z"
}
```

**Mock JSON (lista)**
```json
[
  {
    "id": "tok_20260518_001",
    "tenantId": "tnt_cliente_x",
    "runId": "run_20260518_001",
    "ticketId": "JIRA-123",
    "provider": "OPENAI",
    "model": "gpt-5",
    "inputTokens": 12000,
    "outputTokens": 4500,
    "totalTokens": 16500,
    "estimatedCostUsd": 1.85,
    "createdAt": "2026-05-18T21:35:00Z"
  },
  {
    "id": "tok_20260518_002",
    "tenantId": "tnt_cliente_x",
    "runId": "run_20260518_002",
    "ticketId": "JIRA-124",
    "provider": "OPENAI",
    "model": "gpt-5",
    "inputTokens": 8000,
    "outputTokens": 3200,
    "totalTokens": 11200,
    "estimatedCostUsd": 1.10,
    "createdAt": "2026-05-18T22:05:00Z"
  }
]
```

### Model: BudgetLimit

**Descrizione business**
- Limite di spesa/consumo per tenant con azioni automatiche (alert/stop) al superamento.

**Schema**
- id: string (required) es. `bud_tnt_cliente_x_monthly`
- tenantId: string (required)
- period: string (required) enum: `DAILY|WEEKLY|MONTHLY`
- maxCostUsd: number (required)
- actionOnBreach: string (required) enum: `ALERT_ONLY|STOP_RUNS|DISABLE_AGENT`
- enabled: boolean (required)
- createdAt: string (ISO-8601) (required)
- updatedAt: string (ISO-8601) (required)

**Mock JSON (singolo)**
```json
{
  "id": "bud_tnt_cliente_x_monthly",
  "tenantId": "tnt_cliente_x",
  "period": "MONTHLY",
  "maxCostUsd": 250.0,
  "actionOnBreach": "STOP_RUNS",
  "enabled": true,
  "createdAt": "2026-05-18T21:05:00Z",
  "updatedAt": "2026-05-18T21:05:00Z"
}
```

**Mock JSON (lista)**
```json
[
  {
    "id": "bud_tnt_cliente_x_monthly",
    "tenantId": "tnt_cliente_x",
    "period": "MONTHLY",
    "maxCostUsd": 250.0,
    "actionOnBreach": "STOP_RUNS",
    "enabled": true,
    "createdAt": "2026-05-18T21:05:00Z",
    "updatedAt": "2026-05-18T21:05:00Z"
  },
  {
    "id": "bud_tnt_cliente_x_daily",
    "tenantId": "tnt_cliente_x",
    "period": "DAILY",
    "maxCostUsd": 15.0,
    "actionOnBreach": "ALERT_ONLY",
    "enabled": true,
    "createdAt": "2026-05-18T21:05:00Z",
    "updatedAt": "2026-05-18T21:05:00Z"
  }
]
```

### Model: OffboardingJob

**Descrizione business**
- Processo di fine contratto: epurazione segreti, export audit, archiviazione e marcatura tenant come OFFBOARDED.

**Schema**
- id: string (required) es. `off_20260518_001`
- tenantId: string (required)
- status: string (required) enum: `PLANNED|RUNNING|SUCCEEDED|FAILED`
- steps: { name: string, status: string, startedAt: string, finishedAt?: string, errorCode?: string }[] (required)
- createdAt: string (ISO-8601) (required)
- updatedAt: string (ISO-8601) (required)

**Mock JSON (singolo)**
```json
{
  "id": "off_20260518_001",
  "tenantId": "tnt_cliente_x",
  "status": "SUCCEEDED",
  "steps": [
    { "name": "revoke_and_purge_secrets", "status": "SUCCEEDED", "startedAt": "2026-05-18T22:00:00Z", "finishedAt": "2026-05-18T22:01:00Z" },
    { "name": "export_audit_bundle", "status": "SUCCEEDED", "startedAt": "2026-05-18T22:01:00Z", "finishedAt": "2026-05-18T22:03:00Z" },
    { "name": "archive_vm_data", "status": "SUCCEEDED", "startedAt": "2026-05-18T22:03:00Z", "finishedAt": "2026-05-18T22:10:00Z" }
  ],
  "createdAt": "2026-05-18T21:55:00Z",
  "updatedAt": "2026-05-18T22:10:00Z"
}
```

**Mock JSON (lista)**
```json
[
  {
    "id": "off_20260518_001",
    "tenantId": "tnt_cliente_x",
    "status": "SUCCEEDED",
    "steps": [
      { "name": "revoke_and_purge_secrets", "status": "SUCCEEDED", "startedAt": "2026-05-18T22:00:00Z", "finishedAt": "2026-05-18T22:01:00Z" }
    ],
    "createdAt": "2026-05-18T21:55:00Z",
    "updatedAt": "2026-05-18T22:10:00Z"
  },
  {
    "id": "off_20260501_001",
    "tenantId": "tnt_cliente_y",
    "status": "FAILED",
    "steps": [
      { "name": "revoke_and_purge_secrets", "status": "SUCCEEDED", "startedAt": "2026-05-01T18:00:00Z", "finishedAt": "2026-05-01T18:01:00Z" },
      { "name": "export_audit_bundle", "status": "FAILED", "startedAt": "2026-05-01T18:01:00Z", "errorCode": "STORAGE_UNAVAILABLE" }
    ],
    "createdAt": "2026-05-01T17:55:00Z",
    "updatedAt": "2026-05-01T18:02:00Z"
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

```json
{
  "error": {
    "code": "AUTH_REQUIRED",
    "message": "Autenticazione richiesta",
    "details": []
  }
}
```

```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "Permessi insufficienti",
    "details": [
      { "reason": "TENANT_SCOPE_VIOLATION" }
    ]
  }
}
```

```json
{
  "error": {
    "code": "POLICY_BLOCK",
    "message": "Azione bloccata da policy",
    "details": [
      { "reason": "PENDING_REVIEW_REQUIRED" }
    ]
  }
}
```

```json
{
  "error": {
    "code": "INTEGRATION_ERROR",
    "message": "Errore integrazione esterna",
    "details": [
      { "system": "JIRA", "reason": "WEBHOOK_DELIVERY_FAILED" }
    ]
  }
}
```

```json
{
  "error": {
    "code": "BUDGET_BREACH",
    "message": "Budget superato: esecuzione bloccata",
    "details": [
      { "period": "MONTHLY", "maxCostUsd": 250.0, "action": "STOP_RUNS" }
    ]
  }
}
```

```json
{
  "error": {
    "code": "SNAPSHOT_REQUIRED",
    "message": "Snapshot richiesto prima di procedere",
    "details": [
      { "scope": "POSTGRES", "reason": "DESTRUCTIVE_OPERATION" }
    ]
  }
}
```
