---
title: 'Come installare K3D'
date: 2024-01-16T09:52:46+01:00
draft: false

tags:
 - K3S
 - K3D
 - Kubernetes
 - Rancher
 - 
categories:
 - linux
 - server
 - kubernetes

   
cover:
 image: 
---

Iniziamo con capire che cos'è K3D e non confondiamolo con K3S

**K3D** 

K3D è un "wrapper" che come scrive [Wikipedia](https://it.wikipedia.org/wiki/Wrapper) "è un'avvolgitore, un modulo software che ne "riveste" un altro" Si la traduzione dall'inglese non è felicissima, in questo caso specifico consente di eseguire K3S, che è la distribuzione minimale di Kubernetes sviluppata da Rancher Labs, all'interno di Docker. 

In altre parole, K3D semplifica la creazione e la gestione di cluster Kubernetes leggeri e portatili che utilizzano K3S, rendendo il processo più agevole, specialmente per coloro che sviluppo in locale utilizzando Kubernetes.

È importante notare che K3D è un progetto guidato dalla community, non è un prodotto ufficiale di Rancher (SUSE) che ha creato e mantiene K3S. Tuttavia, K3D offre un'opzione comoda e flessibile per coloro che desiderano utilizzare K3S all'interno di container Docker.

**Prerequisiti**
 - avere a disposizione un server, una VM, un container con distribuzione linux
 - accesso alla rete per scaricare  files
 - curl e/o wget installati
 - avere docker installato, per installarlo vi ricordo un precedente articolo [QUI](https://marcofanuntza.it/posts/come-installare-docker-e-docker-compose-su-ubuntu/)

**Procedura**

Per installare K3D sostanzialmente si tratterà semplicemente di scaricare uno script bash che si occuperà in totale autonomia dell'installazione, abbiamo due alternative, utilizzare curl oppure wget, c'è da dire che lo script utilizzerà curl per scaricare i files quindi curl dovete installarlo comunque.

    sudo apt install curl -y

    sudo wget -q -O - https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
    
    sudo curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

Attendete che lo script completi e poi verificatene il risultato con

    k3d version
      k3d version v5.6.0
      k3s version v1.27.4-k3s1 (default)

Ora che K3D è installato possiamo provare a creare il nostro primo cluster Kubernetes (K3S) eseguendo questo comando

    sudo k3d cluster create mycluster

Attendete il completamento, ci metterà un pò


    INFO[0000] Prep: Network
    INFO[0000] Created network 'k3d-mycluster'
    INFO[0000] Created image volume k3d-mycluster-images
    INFO[0000] Starting new tools node...
    INFO[0001] Creating node 'k3d-mycluster-server-0'
    INFO[0001] Pulling image 'ghcr.io/k3d-io/k3d-tools:5.6.0'
    INFO[0003] Pulling image 'docker.io/rancher/k3s:v1.27.4-k3s1'
    INFO[0003] Starting Node 'k3d-mycluster-tools'
    INFO[0011] Creating LoadBalancer 'k3d-mycluster-serverlb'
    INFO[0012] Pulling image 'ghcr.io/k3d-io/k3d-proxy:5.6.0'
    INFO[0017] Using the k3d-tools node to gather environment information
    INFO[0017] HostIP: using network gateway 172.18.0.1 address
    INFO[0017] Starting cluster 'mycluster'
    INFO[0017] Starting servers...
    INFO[0017] Starting Node 'k3d-mycluster-server-0'
    INFO[0022] All agents already running.
    INFO[0022] Starting helpers...
    INFO[0022] Starting Node 'k3d-mycluster-serverlb'
    INFO[0028] Injecting records for hostAliases (incl. host.k3d.internal) and for 2 network members into CoreDNS configmap...
    INFO[0030] Cluster 'mycluster' created successfully!
    INFO[0030] You can now use it like this:
    kubectl cluster-info
    



