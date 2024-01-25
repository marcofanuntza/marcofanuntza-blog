---
title: 'Proviamo Kubernetes con Kind'
date: 2024-01-25T10:37:46+01:00
draft: false

categories:
 - linux
 - server
 - kubernetes
 - Howto
tags:
 - Docker
 - Kubernetes
 - Kind
 - Cluster

cover:
    image: img/cover-kind.webp
  
---



**Proviamo Kubernetes con Kind**

Questa guida è indicata per tutti coloro che hanno esigenza di interagire con un cluster Kubernetes per meri scopi di test, conoscenza e sviluppo utilizzando una workstation o notebook con risorse limitate.


**Kind (Kubernetes IN Docker)** è uno strumento open-source progettato per semplificare la creazione e la gestione di cluster Kubernetes locali utilizzando container Docker come nodi del cluster. 

Ecco alcune caratteristiche chiave di kind:

**Installazione Semplificata:** kind semplifica notevolmente il processo di installazione di Kubernetes su una macchina locale, consentendo agli sviluppatori di creare rapidamente e facilmente cluster Kubernetes per scopi di sviluppo o test.

**Utilizzo di Docker come Nodi:** kind utilizza container Docker per creare i nodi del cluster Kubernetes. Ogni nodo del cluster viene eseguito come un container Docker specifico, consentendo un'implementazione leggera e isolata del cluster locale.

**Ambienti Isolati:** kind consente agli sviluppatori di creare cluster Kubernetes completamente isolati, garantendo che le risorse e le configurazioni di un cluster non interferiscano con altri cluster o ambienti.

**Configurazione Dichiarativa:** La configurazione di kind è dichiarativa e può essere definita attraverso file di configurazione in stile YAML. Questo approccio semplifica la creazione e la gestione di cluster con configurazioni complesse.

**Integrazione con Strumenti di CI/CD:** kind è spesso utilizzato negli ambienti di sviluppo e nei flussi di lavoro (CI/CD) per testare e validare applicazioni Kubernetes.

**Agilità nello Sviluppo e nel Test:** kind consente agli sviluppatori di eseguire e testare le proprie applicazioni Kubernetes in un ambiente locale, facilitando lo sviluppo, il debug e il test delle applicazioni Kubernetes senza la necessità di un cluster remoto.

**Estendibile e Configurabile:** In quanto strumento open-source, kind è estendibile e può essere configurato per soddisfare esigenze specifiche. Gli sviluppatori possono personalizzare i cluster creati con kind per rispecchiare le configurazioni desiderate.

In sintesi, kind è uno strumento che semplifica l'installazione di cluster Kubernetes locali, facilitando il processo di sviluppo, test e debug delle applicazioni Kubernetes in un ambiente controllato e isolato.



**Prerequisiti**

- Virtual Machine, Workstation o Notebook con distribuzione Linux (si può utilizzare anche WSL2 su Windows con Docker Desktop installati)
- Docker
- Kubectl per interagire con il cluster


**Procedimento**

Per questo esempio io utilizzo una VM installata sul mio cluster Proxmox, la distribuzione utilizzata è Ubuntu 22-04-LTS, iniziamo con installare Docker

    # Aggiungiamo la chiave **GPG key** ufficiale del repository Docker
    sudo apt-get update
    sudo apt-get install ca-certificates curl gnupg
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg
    echo "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io 


Installiamo Kind, sostanzialmente si tratta di scaricare il file binario eseguibile, impostare i giusti permessi e spostarlo sul path riservato ai file eseguibili

     # For AMD64 / x86_64
     [ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64

     sudo chmod +x ./kind
     sudo mv ./kind /usr/local/bin/kind

Installiamo Kubectl, il concetto è lo stesso precedente

     sudo curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
     sudo chmod +x kubectl
     sudo mv kubectl /usr/local/bin

     


Verifichiamo le versioni appena installate di docker, Kind e Kubectl

    docker --version
    Docker version 25.0.1, build 29cf629

    kind --version
    kind version 0.20.0


    kubectl version
    Client Version: v1.29.1
    Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3


Adesso siamo pronti per creare il nostro primo cluster Kubernetes utilizzando Kind, decidiamo prima come deve essere composto il nostro cluster, per esempio per le nostre esigenze desideriamo un cluster con 1 nodo control-plane e 3 nodi worker. Per istruire Kind a fare questo dobbiamo scrivere un file di configurazione.



File esempio in formato yaml, cluster-config.yaml

    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    # patch the generated kubeadm config with some extra settings
    kubeadmConfigPatches:
    - |
      apiVersion: kubelet.config.k8s.io/v1beta1
      kind: KubeletConfiguration
      evictionHard:
        nodefs.available: "0%"
    # patch it further using a JSON 6902 patch
    kubeadmConfigPatchesJSON6902:
    - group: kubeadm.k8s.io
      version: v1beta2
      kind: ClusterConfiguration
      patch: |
        - op: add
        path: /apiServer/certSANs/-
        value: my-hostname
    # 1 control plane node and 3 workers
    nodes:
    # the control plane node config
    - role: control-plane
      image: kindest/node:v1.27.3@sha256:3966ac761ae0136263ffdb6cfd4db23ef8a83cba8a463690e98317add2c9ba72
      kubeadmConfigPatches:
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "ingress-ready=true"
      extraPortMappings:
      - containerPort: 80
        hostPort: 80
        protocol: TCP
      - containerPort: 443
        hostPort: 443
       protocol: TCP
    # the three workers
    - role: worker
      image: kindest/node:v1.27.3@sha256:3966ac761ae0136263ffdb6cfd4db23ef8a83cba8a463690e98317add2c9ba72
    - role: worker
      image: kindest/node:v1.27.3@sha256:3966ac761ae0136263ffdb6cfd4db23ef8a83cba8a463690e98317add2c9ba72
    - role: worker
      image: kindest/node:v1.27.3@sha256:3966ac761ae0136263ffdb6cfd4db23ef8a83cba8a463690e98317add2c9ba72


Il file oltre a dichiarare i tre nodi, 1 control-plane + 3 worker pre-abilita anche l'ingress controller con le porte 80 e 443. Mi raccomando sul file prestate attenzione alla corretta identazione yaml, altrimenti riceverete errore.

Eseguiamo il seguente comando che ci permetterà di creare il cluster con la configurazione desiderata.

    sudo kind create cluster --name kube-kind-test --config cluster-config.yaml

Il comando completerà i task in circa due minuti, questo potrebbe variare in base alla vostra connessione internet, tenete conto che Docker dovrà scaricare le apposite immagini.

Dovreste ritrovarvi con un'output simile

    Creating cluster "kube-kind-test" ...
     ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
     ✓ Preparing nodes 📦 📦 📦 📦
     ✓ Writing configuration 📜
     ✓ Starting control-plane 🕹️
     ✓ Installing CNI 🔌
     ✓ Installing StorageClass 💾
     ✓ Joining worker nodes 🚜
    Set kubectl context to "kind-kube-kind-test"
    You can now use your cluster with:

    kubectl cluster-info --context kind-kube-kind-test

    Thanks for using kind! 😊



A questo punto possiamo utilizzare kind per verificare il cluster appena creato e kubectl per interagire con esso


    sudo kind get clusters
    kube-kind-test


    sudo kubectl get nodes
    NAME                           STATUS   ROLES           AGE     VERSION
    kube-kind-test-control-plane   Ready    control-plane   5m47s   v1.27.3
    kube-kind-test-worker          Ready    <none>          5m21s   v1.27.3
    kube-kind-test-worker2         Ready    <none>          5m27s   v1.27.3
    kube-kind-test-worker3         Ready    <none>          5m26s   v1.27.3


    sudo kubectl get ns
    NAME                 STATUS   AGE
    default              Active   2m
    kube-node-lease      Active   2m
    kube-public          Active   2m
    kube-system          Active   2m
    local-path-storage   Active   2m



    sudo kubectl get pods -A
    NAMESPACE            NAME                                                   READY   STATUS    RESTARTS   AGE
    kube-system          coredns-5d78c9869d-6dgr5                               1/1     Running   0          7m48s
    kube-system          coredns-5d78c9869d-fchdj                               1/1     Running   0          7m48s
    kube-system          etcd-kube-kind-test-control-plane                      1/1     Running   0          8m2s
    kube-system          kindnet-6p787                                          1/1     Running   0          7m44s
    kube-system          kindnet-7cwt7                                          1/1     Running   0          7m45s
    kube-system          kindnet-fx5lg                                          1/1     Running   0          7m39s
    kube-system          kindnet-tcwbt                                          1/1     Running   0          7m48s
    kube-system          kube-apiserver-kube-kind-test-control-plane            1/1     Running   0          8m2s
    kube-system          kube-controller-manager-kube-kind-test-control-plane   1/1     Running   0          8m2s
    kube-system          kube-proxy-cjs8z                                       1/1     Running   0          7m45s
    kube-system          kube-proxy-n97zk                                       1/1     Running   0          7m44s
    kube-system          kube-proxy-qhmvm                                       1/1     Running   0          7m48s
    kube-system          kube-proxy-vzxpr                                       1/1     Running   0          7m39s
    kube-system          kube-scheduler-kube-kind-test-control-plane            1/1     Running   0          8m2s
    local-path-storage   local-path-provisioner-6bc4bddd6b-zzw4d                1/1     Running   0          7m48s


Infine per completare il discorso ingress-controller procederemo con installazione dell'ingress controller NGINX, il deployment verrà eseguito direttamente con kubectl scaricando il file direttamente dai repository ufficiali NGINX

    sudo kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml


Verifichiamo sempre tramite kubectl

    sudo kubectl get deployment -n ingress-nginx

    NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
    ingress-nginx-controller   1/1     1            1           114s

    sudo kubectl get pods -n ingress-nginx
    NAME                                        READY   STATUS      RESTARTS   AGE
    ingress-nginx-admission-create-jk94q        0/1     Completed   0          2m44s
    ingress-nginx-admission-patch-qwc96         0/1     Completed   0          2m44s
    ingress-nginx-controller-864894d997-48cn4   1/1     Running     0          2m44s



Adesso avete a disposizione tutti gli elementi per provare, testare e utilizzare un cluster Kubernetes. La guida termina quì, spero sia stata semplice e chiara da seguire!

Ps. immagine in copertina creata da Dall-E tramite Microsoft Co-pilot  (peccato per il testo :P )
