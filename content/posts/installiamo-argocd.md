---
title: 'Installiamo Argocd'
date: 2024-01-26T09:02:24+01:00
draft: true

categories:
 - linux
 - Howto
 - server

tags:
 - Docker
 - Kubernetes
 - Argo-cd
 - Hem

cover:
  image:
---



Installiamo ARGO-CD


**ARGO CD**

Argo CD è uno strumento open-source progettato per implementare e gestire il CD (continous deployment) su infrastrutture Kubernetes. 
Si basa sui principi GitOps, utilizza repository Git come unica fonte di verità per la configurazione dell'infrastruttura e delle applicazioni. 

**I principali punti chiave di ARGO CD**

**Continuous Deployment:** ARGO CD automatizza il processo di implementazione delle applicazioni su cluster Kubernetes, garantendo che lo stato attuale corrisponda a quello dichiarato nel repository Git.

**GitOps:** Basato sul concetto di GitOps, ARGO CD utilizza un repository Git come fonte unica di verità per la configurazione di infrastruttura e applicazioni.

**Dashboard Intuitiva:** Fornisce un'interfaccia web che consente agli utenti di visualizzare e gestire lo stato delle applicazioni nel cluster Kubernetes.

**Rilasci:** Semplifica i rilasci delle applicazioni tra i diversi ambienti, facilitando il passaggio da sviluppo a produzione attraverso il processo GitOps.

**Rollback:** Permette di tornare a versioni precedenti delle applicazioni in modo controllato in caso di problemi dopo un rilascio.

**Integrazione con Strumenti di CI/CD:** Può essere integrato con strumenti di CI/CD per creare un flusso di lavoro completo dall'implementazione iniziale alla gestione continua delle applicazioni.

**Multi-Cluster e Multi-Tenancy:** Supporta l'implementazione su più cluster Kubernetes e offre funzionalità di multi-tenancy per isolare risorse e permessi.

**Estendibile e Personalizzabile:** Essendo open-source ARGO CD è estendibile, consentendo agli utenti di creare estensioni personalizzate per adattarsi alle esigenze specifiche.



Attraverso queste caratteristiche ARGO CD semplifica e automatizza il ciclo di vita delle applicazioni deployate su Kubernetes.


**Prerequisiti**

- Avere a disposizione un cluster Kubernetes (anche locale)
- Avere una workstation/notebook, il vostro server di sviluppo ad'esempio, dove poter installare quanto segue
- Kubectl
- Helm


Se non avete a disposizione un cluster Kubernetes, per installarlo possiamo seguire molteplici strade, in questo blog ho già scritto alcune guide che potrebbero aiutarvi nel raggiungere lo scopo. Tenete conto che non abbiamo bisogno di un cluster production-grade, basta un semplice cluster locale installato sulla vostra workstation/notebook.

Per questo articolo io ho utilizzato un cluster Kubernetes installato con Kind, potete seguire questa guida [QUI](https://marcofanuntza.it/posts/proviamo-kubernetes-con-kind/)

Ora.. se avete seguito la guida che vi ho indicato più sù, diamo per scontato che abbiamo già il cluster Kubernetes pronto con Kubectl, non ci rimane quindi che installare Helm.

     sudo curl -s https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash


Adesso che abbiamo tutti gli elementi per procedere, iniziamo con creare il namespache da dedicare a argocd sul nostro cluster kubernetes

    sudo kubectl create namespace argocd


L'installazione di Argo-CD verrà eseguita tramite Helm, nello specifico utilizzeremo il repository sviluppato dal team Bitnami, aggiungiamo il repo

    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm repo update

Adattate il comando che segue in base alle vostre esigenze, prestate attenzione alla password e all'hostname del relativo ingress che verrà creato

    helm --install argocd --set config.secret.argocdServerAdminPassword=AB12345 --set server.ingress.enabled=true --set server.ingress.ingressClassName=nginx \
    --set server.service.type=ClusterIP --set server.ingress.pathType=Prefix --set server.ingress.hostname=argocd.marcofanuntza.it bitnami/argo-cd --namespace argocd
