---
title: 'Come installare K3D'
date: 2024-01-16T09:52:46+01:00
draft: false

tags:
 - K3S
 - K3D
 - Kubernetes
 - Rancher
 - installare
 - Ubuntu
categories:
 - linux
 - server
 - kubernetes

   
cover:
    image: /img/k3d-cover.webp
---

Iniziamo con capire che cos'è **K3D** e non confondiamolo con **K3S**

**K3D** 

K3D è un "wrapper" che come scrive [Wikipedia](https://it.wikipedia.org/wiki/Wrapper) "è un'avvolgitore, un modulo software che ne "riveste" un altro" Si la traduzione dall'inglese non è felicissima, in questo caso specifico consente di eseguire K3S, che è la distribuzione minimale di Kubernetes sviluppata da Rancher Labs, all'interno di Docker. 

In altre parole, K3D semplifica la creazione e la gestione di cluster Kubernetes leggeri e portatili che utilizzano K3S, rendendo il processo più agevole, specialmente per coloro che sviluppano in locale utilizzando tecnologie Kubernetes.

È importante notare che **K3D** è un progetto guidato dalla community, non è un prodotto ufficiale di **Rancher (SUSE)** che ha creato e mantiene K3S. Tuttavia, K3D offre un'opzione comoda e flessibile per coloro che desiderano utilizzare K3S all'interno di container Docker.

**Prerequisiti**
 - avere a disposizione un server o una VM con distribuzione linux (si può installare anche su Windows o Mac ma non lo tratteremo quì)
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

Attendete il completamento, nel mio caso ha completato in poco più di 30 secondi


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
    
Ora come suggerisce l'ultima riga dell'output, per interagire con il cluster dobbiamo utilizzare kubectl, installiamolo quindi

    sudo curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    sudo chmod +x kubectl
    sudo mv kubectl /usr/local/bin

Dobbiamo fare in modo che kubectl conosca il file di configurazione del cluster

    sudo k3d kubeconfig merge mycluster --kubeconfig-switch-context

Adesso siamo pronti per interagire con il cluster

    sudo kubectl get nodes -o wide

    NAME                     STATUS   ROLES                  AGE   VERSION        INTERNAL-IP   EXTERNAL-IP   OS-IMAGE   KERNEL-VERSION      CONTAINER-RUNTIME
    k3d-mycluster-server-0   Ready    control-plane,master   12m   v1.27.4+k3s1   172.18.0.2    <none>        K3s dev    5.15.0-91-generic   containerd://1.7.1-k3s1

Come potrete notare il cluster è composto da un unico nodo K3S, ma questo sarà comunque sufficente per eseguire tutti i test e le esigenze di sviluppo.

Continuando a curiosare potete vedere cosa ha installato K3D

    sudo kubectl get svc -A
    
    NAMESPACE     NAME             TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
    default       kubernetes       ClusterIP      10.43.0.1      <none>        443/TCP                      20m
    kube-system   kube-dns         ClusterIP      10.43.0.10     <none>        53/UDP,53/TCP,9153/TCP       20m
    kube-system   metrics-server   ClusterIP      10.43.151.46   <none>        443/TCP                      20m
    kube-system   traefik          LoadBalancer   10.43.61.96    172.18.0.2    80:32526/TCP,443:32411/TCP   19m

    sudo kubectl get pods -A
    
    NAMESPACE     NAME                                     READY   STATUS      RESTARTS   AGE
    kube-system   local-path-provisioner-957fdf8bc-g6vsb   1/1     Running     0          20m
    kube-system   coredns-77ccd57875-h6krb                 1/1     Running     0          20m
    kube-system   metrics-server-648b5df564-nn5kk          1/1     Running     0          20m
    kube-system   helm-install-traefik-crd-52bfs           0/1     Completed   0          20m
    kube-system   helm-install-traefik-nzmhv               0/1     Completed   1          20m
    kube-system   svclb-traefik-caab8633-t7m98             2/2     Running     0          19m
    kube-system   traefik-64f55bb67d-gnvf8                 1/1     Running     0          19m


Personalmente conosco anche un altra alternativa per creare velocemente un cluster kubernetes, forse scriverò un prossimo articolo, l'alternativa cmq è **kind**



