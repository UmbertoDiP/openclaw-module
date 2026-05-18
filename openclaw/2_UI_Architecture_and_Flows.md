---
title: UI Architecture & Flows — OpenClaw
audience: AI Builder
version: 0.1
---

# 2) UI Architecture & Flows — OpenClaw

## 2.1 Mappa Viste (IA)

- Home
- Autenticazione (se applicabile)
- Dashboard
- Lista oggetti principali
- Dettaglio oggetto
- Creazione/Modifica
- Impostazioni

## 2.2 Navigazione (happy path)

1) Home → …
2) Dashboard → …

## 2.3 Componenti UI (macro)

- Layout e navigazione
- Liste / Tabelle
- Form
- Modali / Drawer
- Toast / Alert

## 2.4 Stati UI (obbligatori per ogni vista)

- Loading: cosa mostra e cosa è disabilitato
- Empty: messaggio + CTA
- Success: contenuto + azioni
- Error: messaggio + retry + fallback

## 2.5 Specifica per Vista (template)

### Vista: [Nome Vista]

- Scopo:
- Chi la usa:
- Dati richiesti:
- Azioni principali:
- Validazioni:
- Messaggi (success/error):

**Stati UI**
- Loading:
- Empty:
- Success:
- Error:

## 2.6 Regole UI (derivate da business)

- Abilitazioni/disabilitazioni azioni:
- Visibilità per ruolo:
- Stati read-only:
