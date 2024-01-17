---
title: Proviamo Kubernetes con Rancher
date: 2024-01-17T09:01:55+01:00
draft: true

categories:
 - server
 - linux
 - howto
tags:
 - kubernetes
 - rancher
 - k3d
 - k3s
 - helm

cover:
  img: 
---

Kubernetes è un sistema di gestione (orchestratore) di container che è diventato di fatto lo standard per distribuire applicazioni containerizzate.

Questo perché Kubernetes è potente, affidabile, flessibile e per lo più facile da usare. Si facile da utilizzare dopo che si supera il primo scoglio iniziale.. 

Io personalmente ho avuto difficoltà nel visualizzare mentalmente il cluster e tutti i componenti che ne facevano parte utilizzando solo gli strumenti della riga di comando finché non ho familiarizzato con la sua struttura.

Per fare ciò è stato di grandissimo aiuto (per me) l'utilizzo di **Rancher!**

Con questo articolo spero di aiutare tutti gli interessati che vogliono conoscere e sperimentare Kubernetes, il risultato finale di questa guida vi permetterà di avere una sorta di laboratorio sulla vostra workstation/server e toccare con mano un cluster Kubernetes.

Per raggiungere le scopo verranno installati i seguenti elementi:

**Docker:** Il sistema più diffuso per gestire container
**Kubectl:** Strumento a riga di comando utilizzato per controllare il cluster Kubernetes
**Helm:** Gestore di pacchetti per Kubernetes. Consente di installare, aggiornare e gestire applicazioni su un cluster Kubernetes.
**K3d:** k3d è un progetto guidato dalla community, supportato da Rancher (SUSE). È un wrapper per eseguire k3s in Docker.
**K3s:** È una distribuzione Kubernetes pronta per la produzione, molto leggera, sviluppata da Rancher.
**Rancher:** Banalmente potrebbe essere considerata una GUI per Kubernetes ma fa molto di più! Permette di gestire e configurare più cluster Kubernetes da un unico punto di controllo.

**Premessa**
 - avere a disposizione una workstation o server con distribuzione Linux (si potrebbe utilizzare anche WSL2 su Windows ma non lo tratteremo in questo articolo)
 - accesso alla rete per scaricare pacchetti

**Procedura parte 1**

Qui procediamo con installazione degli elementi Docker, K3D, Helm e Kubectl

Docker

    # Aggiungiamo la chiave **GPG key** ufficiale del repository Docker
    sudo apt-get update
    sudo apt-get install ca-certificates curl gnupg
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    echo "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io 
        
K3D 

     sudo wget -q -O - https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

Helm

    curl -s https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

Kubectl

    sudo curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    sudo chmod +x kubectl
    sudo mv kubectl /usr/local/bin


**Procedura parte 2**

Adesso procederemo con la creazione del cluster Kubernetes tramite K3D utilizzando K3S e infine completeremo installando Rancher tramite Helm.

Creazione cluster Kubernetes

    sudo k3d cluster create k8s-test-rancher -p "8900:30080@agent:0" -p "8901:30081@agent:0" -p "8902:30082@agent:0" --agents 2

Il comando andrà a creare un cluster chiamato "k8s-test-rancher" con 3 porte esposte: 30080, 30081 e 30082. Verranno mappate rispettivamente alle porte 8900, 8901 e 8902 della workstation o server che state utilizzando. Il cluster saà composto da 1 nodo master e 2 nodi worker. 

Ora facciamo in modo che Kubectl conosca il file di configurazione del cluster eseguendo il seguente comando

    sudo k3d kubeconfig merge k8s-test-rancher --kubeconfig-switch-context

Eseguiamo ora una prima verifica della presenza del cluster e dei nodi elencati

    sudo kubectl get nodes

Se tutto procede come dovrebbe dovreste avere lo stesso risultato di sotto







