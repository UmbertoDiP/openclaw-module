---
title: Data Models & Mock Data — OpenClaw
audience: AI Builder
version: 0.1
---

# 3) Data Models & Mock Data — OpenClaw

## 3.1 Convenzioni

- Ogni modello: nome, significato business, campi, vincoli, mock JSON singolo + lista.
- Includere payload errore (validazione/permessi) per guidare stati UI.

## 3.2 Modelli Dati (template)

### Model: [NomeModel]

**Descrizione business**
- 

**Schema**
- id: string (required)
- createdAt: string (ISO-8601) (required)
- updatedAt: string (ISO-8601) (required)

**Vincoli**
- 

**Mock JSON (singolo)**

```json
{
  "id": "obj_001",
  "createdAt": "2026-01-01T10:00:00Z",
  "updatedAt": "2026-01-01T10:00:00Z"
}
```

**Mock JSON (lista)**

```json
[
  {
    "id": "obj_001",
    "createdAt": "2026-01-01T10:00:00Z",
    "updatedAt": "2026-01-01T10:00:00Z"
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
