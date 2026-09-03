# Telegram Data Ingestion & Automation Pipeline

Un backend puro di automazione e integrazione dati sviluppato a budget zero in ambiente locale, progettato per intercettare messaggi di testo non strutturati da Telegram, eseguirne il parsing logico e memorizzare i dati strutturati in un database relazionale.

## 🛠️ Stack Tecnologico & Architettura
*   **Orchestratore/Backend:** n8n (Esecuzione self-hosted in locale)
*   **Database:** PostgreSQL (Data persistence per transazioni storiche)
*   **Ambiente d'Infrastruttura:** Docker Desktop (Isolamento dei servizi e Docker Network dedicata)
*   **Database Client:** Beekeeper Studio (Data inspection e amministrazione schemi)

## 🔄 Flusso Logico dei Dati
Il workflow gestisce due pipeline asincrone in parallelo all'interno dello stesso foglio di lavoro:
1.  **Pipeline di Ingestione (Ogni 5s):** Schedule Trigger ➔ HTTP Request (API `getUpdates` con Polling locale) ➔ Code Node (Parsing JS avanzato per separare cifre e categorie testuali) ➔ PostgreSQL (Inserimento di record incrementali tramite Primary Key autogenerata `SERIAL`).
2.  **Pipeline di Reportistica (Settimanale):** Schedule Trigger ➔ PostgreSQL (Query SQL complessa con aggregazione dati `SUM` e calcolo intervallo temporale `INTERVAL '7 days'`) ➔ HTTP Request (Invio report formattato dinamicamente tramite endpoint `sendMessage` di Telegram).

## 🔒 Considerazioni sulla Sicurezza & Cybersecurity
Il perimetro infrastrutturale è isolato localmente all'interno della rete Docker. In un'ottica di produzione aperta al pubblico, l'applicazione è predisposta per l'implementazione di un **Filtro di Whitelisting** sul `chat_id` all'interno del nodo di elaborazione, impedendo l'inserimento di dati da parte di utenti non autorizzati e prevenendo potenziali attacchi di spoofing o iniezione di record sul database.

## 📈 Scalabilità & Analisi dei Costi (Local vs Cloud TCO)
Progetto originariamente ingegnerizzato in locale per ottimizzazione delle risorse. 
*   **Analisi dei costi Cloud (VPS):** Per una messa in produzione h24, l'infrastruttura prevede la migrazione su una VPS Linux (es. Aruba Cloud / Hetzner) a un costo stimato di ~2.50€/mese, inferiore al consumo elettrico medio di un server domestico hardware (~40W costanti = ~8.50€/mese in Italia).
*   **Ottimizzazione API:** Il passaggio al Cloud consentirà di sostituire il meccanismo di Polling locale con i veri **Webhook HTTPS**, azzerando il carico computazionale della macchina quando il bot è inattivo.

## 👨‍💻 Note di Sviluppo
Ho sviluppato questo progetto per fare un salto dal frontend (Flutter) al backend puro. Volevo capire come manipolare le API senza una GUI. Durante lo sviluppo in locale su Docker ho dovuto combattere con gli errori di HTTPS di Telegram, che ho risolto usando il Polling, e con un errore di chiave primaria duplicata su Postgres perché n8n provava a sovrascrivere l'ID della chat sulla colonna SERIAL
