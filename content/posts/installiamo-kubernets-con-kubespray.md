---
title: "Installare Kubernetes con Kubespray"
date: 2025-02-21
description: "Guida all'installazione di un cluster Kubernetes utilizzando Kubespray, un potente strumento basato su Ansible."
tags: ["Kubernetes", "Kubespray", "Ansible", "Cluster"]
categories: ["DevOps", "Container"]
draft: false
cover:
  image: /img/monit-in.webp

---

## Installare Kubernetes con Kubespray

Installare Kubernetes seguendo la documentazione ufficiale è noto come **"the hard way"** e scoprirete presto quanto questo nome sia appropriato.  
Fortunatamente, grazie all'open source, sono stati sviluppati diversi strumenti per semplificare questa procedura complessa. Uno dei più validi è senza dubbio **Kubespray**.

### Cos'è Kubespray?

**Kubespray** è un progetto open-source che consente di installare e gestire cluster Kubernetes in modo automatizzato tramite **Ansible**.  
È una soluzione altamente flessibile che supporta diversi provider cloud e ambienti on-premises, rendendola adatta a molteplici scenari di deployment.

### Caratteristiche principali

- **Basato su Ansible**: utilizza playbook per l'installazione e la gestione del cluster.
- **Compatibilità multi-cloud e on-premises**: supporta AWS, GCP, Azure, OpenStack, Proxmox, bare metal e altre piattaforme.
- **Alta personalizzazione**: consente di configurare componenti come CNI (*Calico, Flannel, Cilium*), CRI (*containerd, Docker, cri-o*) e molto altro.
- **Alta disponibilità**: permette di creare cluster Kubernetes in modalità *High Availability* con più master e worker.
- **Integrazione con add-on**: supporta strumenti come *Helm, MetalLB, CoreDNS* e altri.

Se state cercando un'alternativa a *kubeadm* che vi offra maggiore controllo sulla configurazione del cluster, **Kubespray è la scelta ideale**.

## Installazione di Kubernetes con Kubespray

In questa guida, vedremo passo dopo passo come installare un cluster Kubernetes utilizzando Kubespray.  


**Prerequisiti**

La mia installazione è stata eseguita utilizzando alcune virtual machine create sul mio cluster Proxmox. Nel dettaglio abbiamo bisogno di:

- 3 VM per i nodi master

- 3 VM per i nodi worker

- workstation con distribuzione linux, oppure  WSL2 se utilizzate Windows.

La lista delle VM e relativi IP:

     kubespray-master-01 192.168.1.184

     kubespray-master-02 192.168.1.189

     kubespray-master-03 192.168.1.190

     kubespray-worker-01 192.168.1.191

     kubespray-worker-02 192.168.1.192

     kubespray-worker-03 192.168.1.193



Su workstation clonare il repository git, io ho utilizzato la versione 1.25.0, creare un "env" ambiente virtuale per pyrhon e installare i moduli "requirements" tramite pip3

    git clone https://github.com/kubernetes-sigs/kubespray.git

    cd kubespray

    python3 -m venv kubespray-venv

    source kubespray-venv/bin/activate

    pip3 install -U -r requirements.txt

Ora è necessario editare i files necessari al deploy tramite ansible, copiamo una directory sample e editiamola in base alle nostre esigenze

    cp -rfp inventory/sample inventory/proxmox01

    declare -a IPS=(192.168.1.184 192.168.1.189 192.168.1.190 192.168.1.191 192.168.1.192 192.168.1.193)

    CONFIG_FILE=inventory/proxmox01/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

Il comando precedente ha creato un file chiamato hosts-yaml e inserito IP, dobbiamo editarlo ulteriormente con i nomi dei nostri nodi

    vim inventory/proxmox01/hosts.yaml

Aggiungiamo un'ulteriore file dove inseriremo alcune variabili

    vi inventory/proxmox01/cluster-variables.yaml

Questo sarà il contenuto:

    kube_version: v1.30.1
    helm_enabled: true
    kube_proxy_mode: iptables

Ora siamo pronti per installare il cluster, prima un'ultima verifica eseguendo un test di raggiungibilità sui nodi, oltre al comando che segue dobbiamo copiare la nostra chiave SSH sui nodi precedentemente creati
il comando che segue eseguirà un ping

    ansible -i inventory/proxmox01/hosts.yaml -m ping all -u service

Se tutti i nodi saranno raggiungibili senza problemi siamo finalmente pronti per installare il cluster

    ansible-playbook -i inventory/proxmox01/hosts.yaml -e @inventory/proxmox01/cluster-variables.yaml --become --become-user=root -u service cluster.yml

Il comando è abbastanza parlante e chiaro, ora attendete che kubespray tramite ansible installi il cluster kubernetes nei nodi predefiniti, dovrebbe impiegare alcuni minuti.
Terminato il tutto dovreste avere il messaggio di recap simile a questo

![Example image](/img/kubespray_02.webp)

verifichiamo il cluster kubernetes, eseguiamo un accesso ssh sul primo nodo master e facciamo in modo che il nostro user possa chiamare il cluster tramite kubectl, per fare questo copiamo il file di configurazione come segue:


    sudo mkdir -p $HOME/.kube

    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

    sudo chown $(id -u):$(id -g) $HOME/.kube/config
 

Ora il più classico dei comandi
 
    kubectl get nodes


![Example image](/img/kubespray_03.webp)




