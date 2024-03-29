---
title: 'Harbor come Registry'
date: 2024-01-31T11:26:58+01:00
draft: false

tags:
- Kubernetes
- Docker
- Harbor
- Registry
- docker registry
- installare

categories:
- Howto
- Kubernetes
- Docker

cover:
  image: /img/cover-harbor.webp
---


**Harbor** è un registry open-source per la gestione delle immagini dei container. Progettato per funzionare con orchestration tools come Kubernetes e Docker Swarm.

Ecco alcune caratteristiche principali di Harbor:

**Harbor** offre un registry sicuro e privato per le immagini dei container, permettendo un controllo totale sulla loro archiviazione e distribuzione.

**Politiche di Sicurezza:** Supporta politiche per garantire che solo immagini sicure e approvate vengano utilizzate nell'ambiente.

**Controllo degli Accessi:** Dispone di un sistema robusto di controllo degli accessi, consentendo la definizione precisa di chi può accedere e distribuire immagini specifiche.

**Integrazione Universale:** Progettato per integrarsi facilmente con strumenti comuni come Kubernetes, Docker e altri.

**Replicazione di Immagini:** Consente la replicazione tra diversi registri Harbor, garantendo la disponibilità e la distribuzione su scala geografica.

**Scansione delle Vulnerabilità:** Fornisce strumenti di scansione per identificare e affrontare potenziali problemi di sicurezza nelle immagini dei container.

**Gestione del Ciclo di Vita:** Supporta l'intero ciclo di vita delle immagini, dalla creazione alla distribuzione e alla dismissione.

Harbor è particolarmente utile in ambienti aziendali in cui la sicurezza e il controllo dell'accesso alle immagini container sono fondamentali. La sua architettura aperta e scalabile lo rende adatto sia a scenari di piccola scala che a implementazioni di grandi dimensioni.

**Proviamolo** installandolo su un server o su una VM

**Prerequisiti**

-VM o server con distribuzione Linux (in questo articolo ho utilizzato ubuntu-server-22-04-LTS)

-Docker

-Docker compose

-Openssl

Per installare Docker e Docker Compose posso segnalarvi questo howto che ho scritto in precedenza [QUI](https://marcofanuntza.it/posts/come-installare-docker-e-docker-compose-su-ubuntu/)

**Procedura**

Dopo aver installato i prerequisti, dobbiamo assicurarci che il server o la VM soddisfino i requisiti consigliati da Harbor, deve avere a disposizione almeno 4GB di Ram e 2CPU, per lo spazio disco possono bastare 20GB, che comunque possono essere estesi al bisogno se avete utilizzato LVM.

L'installazione di Harbor viene eseguita tramite un'installer che possiamo scaricare dal sito ufficiale, noi utilizzeremo la versione 2.10.0

    wget https://github.com/goharbor/harbor/releases/download/v2.10.0/harbor-online-installer-v2.10.0.tgz

    tar -xzvf harbor-online-installer-v2.10.0.tgz

    cd harbor
    
    cp harbor.yml.tmpl harbor.yml

a questo punto dobbiamo creare il nostro certificato SSL, per questa guida inizialmente utilizzeremo un certificato self-signed creato con openssl

    mkdir -p certs
    openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/harbor.key -addext "subjectAltName = DNS:harbor.marcofanuntza.it" -x509 -days 365 -out certs/harbor.crt

dopo aver terminato copiamo il risultato sul path dei certificati

    sudo cp certs/harbor.* /etc/ssl/certs/


editiamo il file harbor.yml che abbiamo copiato in precedenza, dobbiamo inserire hostname e il path del certificato SSL

    hostname: harbor.marcofanuntza.it

    # http related config
    http:
        # port for http, default is 80. If https enabled, this port will redirect to https port
        port: 80

    https:
       # https port for harbor, default is 443
       port: 443
    # The path of cert and key files for nginx
       certificate: /etc/ssl/certs/harbor.crt
       private_key: /etc/ssl/certs/harbor.key



adesso sempre all'interno della directory **harbor** possiamo notare due script bash, il primo da eseguire sarà prepare e in seguito eseguiremo install.sh

    ./prepare

lo script prepare scaricherà una prima immagine docker da docker-hub, questa preparerà una base per lo script successivo che di fatto andrà a tirare sù tutto lo stack docker tramite docker compose

    ./install.sh

mettetevi comodi perche ci vorrà un pò, verranno scaricate le immagini docker per redis, postresql, registry, proxy, core e portal, successivamente verranno tutte avviate in automatico, dovete attendere un output simile a quanto segue

     ✔ Container harbor-log         Started 
     ✔ Container harbor-portal      Started 
     ✔ Container registry           Started
     ✔ Container redis              Started 
     ✔ Container registryctl        Started 
     ✔ Container harbor-db          Started 
     ✔ Container harbor-core        Started 
     ✔ Container harbor-jobservice  Started 
     ✔ Container nginx              Started 


sembra che tutto sia andato per il verso giusto, per scrupolo comunque potete verificare che tutti i container siano in stato di esecuzione tramite docker ps


    CONTAINER ID   IMAGE                                 COMMAND                  CREATED         STATUS                        PORTS                                                                            NAMES
    a069f83f6725   goharbor/harbor-jobservice:v2.10.0    "/harbor/entrypoint.…"   3 minutes ago   Up About a minute (healthy)                                                                                    harbor-jobservice
    e96dfe0dc5a0   goharbor/nginx-photon:v2.10.0         "nginx -g 'daemon of…"   3 minutes ago   Up 2 minutes (healthy)        0.0.0.0:80->8080/tcp, :::80->8080/tcp, 0.0.0.0:443->8443/tcp, :::443->8443/tcp   nginx
    89a0f9fc181b   goharbor/harbor-core:v2.10.0          "/harbor/entrypoint.…"   4 minutes ago   Up 2 minutes (healthy)                                                                                         harbor-core
    b0ed82f07593   goharbor/registry-photon:v2.10.0      "/home/harbor/entryp…"   4 minutes ago   Up 3 minutes (healthy)                                                                                         registry
    edabed017c3d   goharbor/harbor-db:v2.10.0            "/docker-entrypoint.…"   4 minutes ago   Up 3 minutes (healthy)                                                                                         harbor-db
    d83c114bbebd   goharbor/harbor-registryctl:v2.10.0   "/home/harbor/start.…"   4 minutes ago   Up 3 minutes (healthy)                                                                                         registryctl
    14fc36784402   goharbor/harbor-portal:v2.10.0        "nginx -g 'daemon of…"   4 minutes ago   Up 3 minutes (healthy)                                                                                         harbor-portal
    7f759eda6cb2   goharbor/redis-photon:v2.10.0         "redis-server /etc/r…"   4 minutes ago   Up 3 minutes (healthy)                                                                                         redis
    1d24113d3097   goharbor/harbor-log:v2.10.0           "/bin/sh -c /usr/loc…"   4 minutes ago   Up 3 minutes (healthy)        127.0.0.1:1514->10514/tcp                                                        harbor-log


Si la nostra installazione di Harbor sembra completata senza intoppi, siamo pronti per accedere alla sua interfaccia web, apriamo il broswer e inseriamo la credenziali admin, la password la trovate in chiaro sul file harbor.yml, cambiatela dopo il primo login


![Example image](/img/harbor-1.webp#center)

ecco la pagina principale

![Example image](/img/harbor-2.webp#center)


Adesso non ci resta che creare gli user e relativi permessi per poter così iniziare a caricare immagini sui rispettivi project!
