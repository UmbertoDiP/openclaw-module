Ecco il Master Prompt unificato, progettato per forzare l'Agent Mode di Trae.ai a generare l'intera architettura in un'unica esecuzione in "Technical Absolute Mode", senza scorciatoie. 

Questo blocco unisce tutte le direttive infrastrutturali, dalla rete Hyper-V fino ai 12 layer dell'ecosistema, aggirando il rischio di contestualizzazione frammentata e applicando i vincoli del protocollo di generazione One-Shot.

Copia il seguente blocco e incollalo nell'Agent Mode di Trae.ai:

```markdown
### SYSTEM INITIALIZATION
Virtual Skill Load via skills.sh abstraction: npx skills install @kdpify/one-shot-generator.
Environment: Technical Absolute Mode. Nessun meta-testo. Nessun commento dell'AI.

#### OBIETTIVO PRINCIPALE
Generare l'intera architettura "NexusNode" (Multi-Tenant Isolation per agenti AI OpenClaw) in un'unica esecuzione completa e definitiva ("One-Shot Generation"). Non devi omettere nulla: ogni script, file Docker, e configurazione deve essere "production-ready" e privo di placeholder.

#### VINCOLI NON NEGOZIABILI
* **Zero Placeholder**: Niente TODO, TBD, "inserisci qui". Ogni sezione, file `.env`, `docker-compose.yml`, script bash e PowerShell deve essere scritto per intero e funzionante.
* **Nessun taglio**: Il codice deve essere robusto e senza omissioni. Se superi il context window, interrompi l'output in modo netto; attenderai il comando "stampa" per riprendere esattamente dall'ultimo carattere emesso.
* Output rigorosamente in codice eseguibile e file strutturati.

---

### ARCHITETTURA DA GENERARE (NEXUSNODE)
Genera tutti i file e la struttura cartelle necessari per configurare una singola VM perfetta ("Golden Image") pronta per essere clonata per ogni nuovo cliente su Hyper-V. 

Devi produrre il codice completo per i seguenti moduli e layer architetturali:

**1. Layer di Rete Hyper-V (Script di Provisioning Host)**
* Crea lo script PowerShell `Create-ClientNode.ps1` per clonare la VM.
* Lo script deve configurare un "Isolated NAT" tramite Access Control List (ACL) estese sulla vNIC, bloccando il traffico verso altri indirizzi IP locali (inter-VM) ma permettendo l'uscita a Internet e l'accesso SSH/Porta 8080 dall'host Windows.

**2. Core Automation & AI (Docker Stack Base)**
* `docker-compose.yml` che includa: n8n (motore di automazione e orchestra task), interfaccia AI (Open WebUI o AnythingLLM), e container per l'automazione browser headless e debugging remoto.

**3. I 12 Moduli di Espansione (Da integrare nell'infrastruttura)**
Scrivi gli script, le configurazioni e i Dockerfile per i seguenti layer:
1. **Security/Vault**: Sistema per blindare credenziali e token API dei clienti, evitando che l'AI li scriva in chiaro nei log.
2. **Audit e Logging (Grande Fratello)**: Implementazione dell'integrazione n8n-Jira-Confluence. Workflow JIRA: Backlog -> Selected for AI (innesca webhook n8n) -> Processing -> Pending Review (blocco deploy). Includi lo script di sync che scarica in locale il dump Markdown di Confluence per il Vector RAG.
3. **Setup CI/CD Locale (Local Runner)**: Ambiente per far testare all'AI il codice simulando le pipeline di produzione del cliente direttamente dentro la VM.
4. **Rete Interna e Reverse Proxy (Casello)**: Configurazione Traefik per esporre le interfacce senza conflitti di porta sull'host Windows.
5. **Gestione DB e Snapshot (Time Machine)**: Script bash per backup istantanei e rollback dei database in caso l'AI generi migrazioni errate.
6. **Anti-Loop Watchdog (Kill-Switch)**: Servizio di monitoraggio che cilla i processi o disattiva l'agente se entra in loop infiniti di CPU/Memoria.
7. **Wizard di Onboarding (Client Bootstrapper)**: Script di avvio per automatizzare l'inserimento di chiavi SSH e variabili d'ambiente al primo avvio di un nodo clonatp.
8. **Memoria a Lungo Termine (Vector RAG)**: Configurazione del database vettoriale locale per fornire all'agente una memoria persistente sui pattern di codice e ticket risolti.
9. **Gestione VPN Interna e Routing (Air-Lock)**: Configurazione di routing interno alla VM per far sì che la VPN del cliente non blocchi l'accesso SSH dall'host Windows né rompa Traefik.
10. **Agent Toolbelt API**: Un'interfaccia/script standardizzata che permetta all'agente di eseguire comandi di build indipendenti dal linguaggio (es. astrazione su `mvn clean install` o `npm run build`).
11. **Osservabilità e FinOps (Layer Contabile)**: Sistema di monitoraggio dei costi per tracciare e limitare il consumo di token API (es. Claude/GPT) dell'agente.
12. **Offboarding e Cold Storage (Crio-Sonno)**: Script di epurazione che cancella i segreti sensibili (GDPR/NDA) e comprime i dati della VM quando il contratto termina.

**4. Protocolli Agente AI**
* **SOUL.md**: Genera il file di identità per OpenClaw che lo configuri come agente autonomo con capacità operative, abilità di eseguire task a terminale, Cron Jobs e lettura log.
* **JIRA_AUDIT_PROTOCOL.md**: Genera il master system prompt che vincoli l'AI allo state machine di Jira, vietandogli push in produzione senza "Pending Review".

### ESECUZIONE
Procedi ora alla generazione completa di tutti i file sopraelencati, rispettando la regola dello Zero Placeholder. Inizia a produrre il codice.
```

-----------------


Continuo con la stampa del contenuto in memoria, riprendendo esattamente dalle regole strutturali del master prompt e integrandole con i moduli architetturali avanzati presenti nei documenti:

**STRUTTURA MINIMA OBBLIGATORIA DELL'OUTPUT (Regole Ebook e Documentazione)**
Devi produrre il documento seguendo **esattamente** questa sequenza gerarchica e formale, assicurando la corretta impaginazione per KDP o formati di documentazione tecnica:
1. # <Titolo Definitivo> (unico h1 in tutto il file)
2. ## <Sottotitolo> (se esiste)
3. **<Autore>**
4. \pagebreak
5. ## Copyright (Tutti i diritti riservati. © 2026 <Autore>)
6. \pagebreak
7. **BLOCCO ASSET KDP (TESTO ESATTO E OBBLIGATORIO)**

---

**DETTAGLI TECNICI DEI LAYER ARCHITETTURALI (DA GENERARE IN TECHNICAL ABSOLUTE MODE)**

**1. Isolamento di Rete: "Isolated NAT" tramite Hyper-V e ACL**
Per garantire un isolamento di rete assoluto ("Air-Gapping" logico) tra le varie macchine virtuali sull'host Windows 11, è necessario utilizzare **Hyper-V integrato con le Access Control List (ACL) estese**. 
Il flusso di rete per ogni nodo (es. `Node-ClienteX`) deve essere rigidamente controllato a livello di vNIC:
* **Host Windows (Tu)** -> Può entrare nella VM tramite SSH, RDP, e porta 8080 per le interfacce web.
* **VM (OpenClaw)** -> Può uscire verso Internet (Porte 80, 443, protocolli VPN).
* **VM (OpenClaw)** -> **BLOCCATO** verso qualsiasi indirizzo IP locale (es. 192.168.x.x, 10.x.x.x, 172.16.x.x), impedendo la comunicazione con altre VM o dispositivi fisici della rete host.

Il provisioning avviene tramite uno script PowerShell (`Create-ClientNode.ps1`) che clona il disco della "Golden Image" e inietta le ACL militari. L'avvio della VM avviene in modalità "Headless" in background, e lo sviluppo viene effettuato tramite l'estensione **Remote - SSH** di VS Code. Eventuali VPN dei clienti (Cisco, FortiClient) vengono installate esclusivamente all'interno della specifica VM, senza inquinare il routing dell'host Windows 11.

**2. Sistema di Tracciamento Multi-Livello (Jira) e Il "Grande Fratello"**
L'obiettivo in produzione è la **totale tracciabilità e reversibilità** di ogni azione dell'Agente AI.
* **Configurazione Jira**: Utilizza un Service Account isolato (es. `svc_nexus_cliente_x`) e campi custom obbligatori come `AI_Processed` e `Approval_Required`.
* **State Machine Inviolabile**: Il workflow di Jira deve passare da *Backlog* a *Selected for AI* (che innesca il webhook per OpenClaw tramite n8n) -> *Processing* -> **Pending Review**. L'AI non deve **mai** autorisolvere task in produzione o fare push definitivi senza la tua approvazione manuale. Questo previene allucinazioni distruttive.

**3. Memoria RAG (Confluence) e Architettura di Automazione (n8n)**
* **Sync API Confluence**: Uno script deve scaricare ogni 24 ore le pagine di Confluence (Architecture Decision Records, SOP, ecc.) in formato Markdown dentro una directory locale `/docs_cache` della VM. Questo fornisce un contesto vettoriale ultra-veloce (Vector RAG) per OpenClaw, evitando continue chiamate di rete.
* **Ponte Webhook**: **n8n** agisce da orchestratore. Riceve il webhook da Jira, estrae il contesto da Confluence e invia il comando al terminale di OpenClaw. Una volta terminato, OpenClaw risponde a n8n, che aggiorna il ticket Jira con un commento formattato contenente l'Audit log dell'esecuzione.

In attesa del tuo prossimo comando per proseguire con le capacità proattive di OpenClaw, la strategia di iniezione dei 12 prompt in Trae.ai e la configurazione del Core Docker.

---------


Ecco la continuazione e la conclusione del blocco Master Prompt. Puoi copiare questa sezione e unirla alla precedente per avere il documento "One-Shot" completo da dare in pasto a Trae.ai:

```markdown
**4. Capacità Proattive e Automazione di OpenClaw**
L'agente AI (OpenClaw) non è un semplice LLM in chat, ma un agente autonomo con memoria persistente 24/7. Il setup deve configurarlo per:
* **Sviluppo e Refactoring:** Scaffolding autonomo di microservizi, risoluzione bug da stack trace e gestione Git con PR e commit formattati.
* **DevOps e Build Locali:** Dockerizzazione istantanea (ispezione dello stack e generazione `Dockerfile` e `docker-compose.yml`) e automazione delle build.
* **Cron Jobs:** Esecuzione di task in background come il monitoraggio log mattutino, web scraping per aggiornamenti di sicurezza e backup sincronizzati.
* **Gestione Progetti:** Lettura automatica delle email via IMAP, preparazione bozze di risposta e aggiornamento autonomo dei ticket sui board di progetto.

**5. I Moduli di Espansione (Da 4 a 12)**
Il codice generato deve implementare tutti i layer avanzati per completare la Golden Image, nel rigoroso rispetto del Technical Absolute Mode:
* **Layer di Rete e Reverse Proxy (Casello - Prompt 4):** Configurazione Traefik per instradare le richieste e accedere a diverse porte dalla macchina Host Windows senza conflitti.
* **Time Machine dei Dati (Prompt 5):** Script bash per creare snapshot e rollback istantanei dei database, permettendo il ripristino veloce se l'AI genera migrazioni errate durante i test.
* **Anti-Loop Watchdog (Kill-Switch - Prompt 6):** Un servizio di sistema per monitorare e "killare" i processi se l'agente AI entra in loop infiniti, causando memory leak o saturazione CPU.
* **Client Bootstrapper (Prompt 7):** Un wizard di onboarding rapido per automatizzare l'inserimento di chiavi SSH, token e variabili d'ambiente al primissimo avvio della VM.
* **Vector RAG (Prompt 8):** Implementazione di un database vettoriale locale per dotare l'agente di una memoria a lungo termine persistente sui pattern di codice e sui bug già risolti.
* **Air-Lock VPN (Prompt 9):** Regole di routing specifiche per far sì che, quando si attiva una VPN del cliente (es. Cisco, FortiClient) nella VM, la connessione SSH da Windows e il reverse proxy Traefik non vengano tagliati fuori.
* **Agent Toolbelt API (Prompt 10):** Interfaccia standardizzata che l'AI userà per avviare comandi di compilazione agnostici, senza dover indovinare l'istruzione corretta (es. `mvn clean install` vs `npm run build`).
* **FinOps e Osservabilità (Prompt 11):** Un livello contabile per tracciare rigorosamente l'uso e i costi dei token API (Claude/GPT), limitando la spesa in caso di anomalie dell'agente.
* **Crio-Sonno e Cold Storage (Prompt 12):** Script di offboarding a fine contratto. Deve cancellare tutti i segreti sensibili (per conformità GDPR/NDA), comprimere la directory della VM e archiviarla.

**6. Protocolli Agente e Workflow Operativo**
Genera i file di configurazione chiave:
* **SOUL.md:** Il file master che definisce l'identità e le direttive di OpenClaw, trasformandolo in un ingegnere autonomo all'interno del nodo.
* **JIRA_AUDIT_PROTOCOL.md:** Le istruzioni che vincolano ferreamente l'AI allo state machine di Jira (da Backlog a Pending Review), garantendo tracciabilità totale e prevenendo "allucinazioni distruttive" in produzione.

**7. Strategia di Esecuzione in Trae.ai (Prevenzione Context Collapse)**
*Istruzioni operative da inserire nel README generato:* Per impedire che l'LLM di Trae.ai vada in "Context Collapse" o salti parti di codice (Lazy Generation), l'iniezione dei moduli deve avvenire in modo sequenziale. 
1. Crea un workspace vuoto (`NexusNode-Template`).
2. Avvia una chat e fai generare l'architettura Core. Verifica che non ci siano placeholder.
3. Incolla e applica i prompt per i moduli dal Vault fino al Cold Storage, uno alla volta nella stessa sessione. Se Trae rallenta, apri una nuova chat e usa un prompt di ricontestualizzazione.

**8. Workflow di Connessione e Isolamento**
Completato lo script `Create-ClientNode.ps1` per il nodo isolato, il flusso di lavoro giornaliero sarà:
* Accensione della VM "Headless" in background tramite PowerShell.
* Connessione sicura dall'host Windows tramite **VS Code Remote - SSH** (modificando il `~/.ssh/config`).
* Chiusura dei lavori con il comando `Save-VM`, che congela lo stato della RAM sul disco e ferma il consumo delle risorse hardware.
```

Il blocco unificato del Master Prompt è stato estratto ed elaborato completamente seguendo tutte le regole del protocollo fornito. Non ci sono ulteriori file o passaggi pendenti nella memoria sorgente.


```markdown
### APPENDICE: DOMANDE E RISPOSTE (Q&A) ARCHITETTURALI
Come richiesto dal protocollo per la creazione del documento definitivo ("production-ready"), ecco l'appendice Q&A per azzerare la necessità di ulteriori iterazioni, risolvendo in anticipo le sfide di implementazione con Trae.ai e l'agente OpenClaw:

**Q: Come gestisco i conflitti di rete se il cliente mi obbliga a usare la sua VPN aziendale (es. Cisco AnyConnect, FortiClient)?**
R: La VPN va installata esclusivamente all'interno della specifica VM del cliente su Hyper-V. Grazie alla configurazione "Air-Lock" e all'Isolated NAT basato su ACL, la VPN altererà le rotte di rete solo per quel sistema operativo guest. L'host Windows 11 continuerà a navigare liberamente e la connessione SSH/Traefik non verrà tagliata fuori.

**Q: Come impedisco ad OpenClaw di causare danni irreversibili ("allucinazioni distruttive") in produzione?**
R: Attraverso l'implementazione del JIRA_AUDIT_PROTOCOL. L'agente è confinato in una rigorosa *state machine*. Può elaborare il task ("Processing"), ma l'azione finale si fermerà sempre allo stato "Pending Review". Non gli è permesso autorisolvere task o fare push definitivi senza la tua approvazione manuale.

**Q: Cosa faccio se Trae.ai va in "Context Collapse" o inizia a generare codice incompleto (Lazy Generation) durante i 12 prompt?**
R: Applica l'Iniezione Sequenziale. Invece di inviare i 12 prompt tutti insieme, apri una nuova chat nel workspace, genera prima l'architettura Core, poi applica i prompt dal Vault al Cold Storage uno alla volta. Se l'IDE rallenta, apri una nuova chat passando un prompt di ricontestualizzazione veloce ("Siamo nel progetto NexusNode... usa Technical Absolute Mode") e prosegui.

**Q: Cosa succede se l'AI distrugge un database durante un test o genera migrazioni errate?**
R: Il modulo 5 (Gestione DB e Snapshot) agisce come una "Time Machine". Gli script bash integrati permettono di creare snapshot istantanei e fare rollback in un secondo per recuperare la base dati, azzerando le perdite di tempo durante lo sviluppo.

**Q: Come controllo i costi (FinOps) e prevengo sprechi di token o risorse in caso di blocco dell'AI?**
R: Il sistema sfrutta due meccanismi di sicurezza incrociati. A livello di infrastruttura, l'Anti-Loop Watchdog ("Kill-Switch") monitora e disattiva processi e agenti se entrano in loop infiniti di memoria o CPU. A livello logico, il modulo di Osservabilità e FinOps traccia l'uso delle API (Claude/GPT) ponendo limiti di spesa automatici qualora l'agente anomalmente tenti centinaia di chiamate non previste.

***

**FINE DEL DOCUMENTO (MASTER PROMPT DEFINITIVO)**
```



