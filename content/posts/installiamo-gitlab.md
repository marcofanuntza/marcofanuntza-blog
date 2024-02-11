---
title: 'Installiamo Gitlab'
date: 2024-02-11T10:59:48+01:00
draft: false

categories:
 - Howto
 - Linux

tags:
 - Gitlab
 - CI/CD
 - Kubernetes
 - Pipeline
 - Ubuntu
 - Git


cover:
  image: 
---

Questo articolo continua la serie denominata "Il potere CI/CD", in precedenza abbiamo mostrato come installare ArgoCD, poi siamo passati al registry con Harbor, adesso è arrivato il momento di Gitlab.


Che cos'è Gitlab?

GitLab è una piattaforma per la gestione del software basata su Git, fornisce un vasto set di strumenti per favorire la collaborazione, automatizzare processi e monitorare lo sviluppo del software durante il suo ciclo di vita.


Ecco alcuni aspetti chiave di GitLab:

**Repository Git:** Fornisce repository Git per il controllo delle versioni del codice sorgente del software.

**Gestione del Progetto:** Include strumenti per la gestione delle attività, la pianificazione, la tracciabilità dei problemi e altro ancora.

**Integrazione Continua e Rilascio Continuo (CI/CD):** Supporta la creazione, il test e il rilascio automatici del software attraverso pipeline CI/CD.

**Controllo degli Accessi e Autorizzazioni:** Consente la definizione precisa dei permessi di accesso per garantire una collaborazione sicura.

**Issue Tracking e Kanban Boards:** Offre strumenti avanzati per la gestione delle problematiche e la visualizzazione del lavoro in corso attraverso Kanban boards.

**Collaborazione e Commenti:** Favorisce la collaborazione tra membri del team con funzionalità di commento e comunicazione integrate.

**Wiki e Documentazione:** Include un sistema di wiki e strumenti per la documentazione, consentendo la creazione e la condivisione di informazioni.

**Registri e Monitoraggio:** Fornisce funzionalità per la registrazione delle attività, il monitoraggio delle performance e la gestione dei log.

**Integrazioni:** Si integra con una varietà di strumenti di sviluppo di terze parti per una maggiore flessibilità.

GitLab è disponibile in diverse edizioni, tra cui una versione gratuita e open source, GitLab Community Edition (CE), e una versione a pagamento, GitLab Enterprise Edition (EE), che offre funzionalità avanzate e supporto aziendale.

Prerequisiti

-VM o server con distribuzione Ubuntu 22.04
-Accesso alla rete
-Sul file hosts inserite il nome che esporrà il servizio, "EXTERNAL_URL" nel mio caso sarà: gitlab.marcofanuntza.it


Procedura

Per installazione dei pacchetti utilizzeremo il sistema APT, partiamo con installare alcune dipendenze con questo comando che segue

    sudo apt-get install -y curl openssh-server ca-certificates tzdata perl postfix

per la parte postfix vi si presenterà una finestra, lasciate "Internetes Site" e procedete con invio.


Per l'installazione di Gitlab sarà necessario utilizzare il loro repository specifico, recuperiamolo tramite curl

    curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash

Adesso che abbiamo a disposizione il repository ufficiale possiamo procedere con installazione tramite APT, nel comando possiamo già da ora settare la variabile EXTERNAL_URL

    sudo EXTERNAL_URL="https://gitlab.marcofanuntza.it" apt-get install gitlab-ee

Nota.1 come avrete potuto notare stiamo installando il pacchetto *-ee che sta per enterprise edition, non preoccupatevi, recentemente gitlab distribuisce un solo pacchetto senza più distinguerlo come faceva in precedenza con la versione CE community edition e enterprise edition EE appunto, la gestione della licenza è facoltativa e successiva per chi ne avrà l'esigenza, il pacchetto sarà comunque lo stesso.

Nota.2 durante installazione in maniera automatica cercherà di crearvi un certificato lets'encrypt, se la vostra "EXTERNAL_URL" non è già esposta su dns pubblici e non sarà raggiungibile questa procedurà fallirà, ma non preoccupatevi l'installazione verrà comunque completata.

Attendiamo che l'installazione tramite APT verrà completata, oltre a gitlab verranno installati vari servizi, ad esempio anche nginx e postgres. Terminata installazione, errore citato su nota 2 a parte, siamo pronti per chiamare il servizio tramite l'interfaccia web.

Le credenziali per il login saranno root e la password temporanea potete recuperarla in chiaro sul file /etc/gitlab/initial_root_password

Dopo il primo login manco a dirlo procediamo subito con la sostituzione della nostra password di root, dopo averla sostituita ci ritroveremo nella pagina iniziale pronti per eseguire un nuovo login con le nuove credenziali.


A questo punto se avete l'esigenza di tenere esposto in rete il servizio non posso che consigliarvi di configurare il certificato SSL (per chi ha avuto l'errore in fase di installazione)
