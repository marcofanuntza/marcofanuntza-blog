---
title: 'Installiamo Argocd'
date: 2024-01-26T09:02:24+01:00
draft: false

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
    image: /img/argocd-4.webp
---




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

**Procedimento**

Ora.. se avete seguito la guida che vi ho indicato più sù, diamo per scontato che abbiamo già il cluster Kubernetes pronto con Kubectl, non ci rimane quindi che installare Helm.

    sudo curl -s https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

    helm version
    version.BuildInfo{Version:"v3.14.0", GitCommit:"3fc9f4b2638e76f26739cd77c7017139be81d0ea", GitTreeState:"clean", GoVersion:"go1.21.5"}


Adesso che abbiamo tutti gli elementi per procedere, iniziamo con creare il namespace da dedicare a argocd sul nostro cluster kubernetes

    sudo kubectl create namespace argocd


L'installazione di Argo-CD verrà eseguita tramite Helm, nello specifico utilizzeremo il repository sviluppato dal team Bitnami, aggiungiamo il repo

    sudo helm repo add bitnami https://charts.bitnami.com/bitnami
    sudo helm repo update

    Hang tight while we grab the latest from your chart repositories...
    ...Successfully got an update from the "bitnami" chart repository
    Update Complete. ⎈Happy Helming!⎈

Adattate il comando che segue in base alle vostre esigenze, prestate attenzione alla password e all'hostname

    sudo helm install argocd --set config.secret.argocdServerAdminPassword=AB12345 --set server.ingress.enabled=true --set server.ingress.ingressClassName=nginx --set server.service.type=ClusterIP --set server.ingress.pathType=Prefix --set server.ingress.hostname=argocd.marcofanuntza.it bitnami/argo-cd --namespace argocd

Riceverete questo output:

    NAME: argocd
    LAST DEPLOYED: Fri Jan 26 08:33:07 2024
    NAMESPACE: argocd
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    CHART NAME: argo-cd
    CHART VERSION: 5.5.0
    APP VERSION: 2.9.5

    ** Please be patient while the chart is being deployed **

    1. Access your Argo CD installation:
    Connect to one of the following hosts:

    http://argocd.marcofanuntza.it

    2. Execute the following commands to obtain the Argo CD credentials:

    echo "Username: \"admin\""
    echo "Password: $(kubectl -n argocd get secret argocd-secret -o jsonpath="{.data.clearPassword}" | base64 -d)"

Verifichiamo per sicurezza se la password rispecchia quanto abbiamo impostato in fase di installazione

    sudo echo "Password: $(sudo kubectl -n argocd get secret argocd-secret -o jsonpath="{.data.clearPassword}" | base64 -d)"
    Password: AB12345

Ok perfetto, possiamo andare avanti. 

Adesso dovete prestare attenzione a questi passaggi perche saranno essenziali per permetterci di raggiungere la Web-Gui di Argo-CD.

Iniziamo con provare a chiamare sul browser la url impostata in fase di installazione, riceverete il warning sul certificato che è normale essendo self-signed, accettate e andate avanti, dovreste ricevere lo stesso errore che segue

![Example image](/img/argocd-1.webp)

Perchè succede questo? Il problema è che di default Argo-CD gestisce la terminazione TLS in autonomia e reindirizza sempre le richieste HTTP in HTTPS. 

Noi abbiamo l'ingress controller che gestisce la terminazione TLS e comunica sempre con il servizio backend tramite HTTP, il risultato è che il server di Argo-CD risponderà sempre con un reindirizzamento in HTTPS. Da quì il nostro errore! 

Una delle soluzioni consiste nel disabilitare HTTPS su Argo-CD, che possiamo impostare utilizzando il flag --insecure sul deployment argocd-server.

Questo problema è effettivamente documentato qui: [link documentazione](https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/#option-2-multiple-ingress-objects-and-hosts)

Modifichiamo quindi il nostro deployment

    sudo kubectl edit deployment argocd-argo-cd-server -n argocd

Editiamo solo questa sezione del file inserendo l'istruzione --insecure, salvate e chiudete il file, Kubernetes in automatico eseguirà un restart del deployment

          containers:
          - args:
             - argocd-server
             - --insecure
             - --staticassets
             - /opt/bitnami/argo-cd/app


Adesso non dovreste più avere l'errore precedente e si presenterà la pagina per il login sulla web-gui di Argo-CD

![Example image](/img/argocd-2.webp)

Proviamo un login utilizzando lo user admin e la password impostata in fase d'installazione

![Example image](/img/argocd-3.webp)

L'installazione di ARGO-CD a questo punto possiamo considerarla conclusa, ora non ci (vi) resta che provare lo strumento, per fare questo abbiamo bisogno però di una applicazione e di un server git/gitlab da interrogare.. sarà l'oggetto del mio prossimo post su questo blog?


**Contenuto Extra**

Dimenticavo che sarebbe altrettanto utile avere la CLI di Argo-CD, quindi installiamola! Possiamo installare la CLI sul notebook o sulla stessa macchina dove abbiamo già installato Helm, oppure dove preferite a patto che Argo-CD sia raggiungibile.

     sudo curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
     sudo chmod +x /usr/local/bin/argocd

     argocd version
     argocd: v2.8.9+3ef5ba7
     BuildDate: 2024-01-19T18:33:48Z
     GitCommit: 3ef5ba76a95d71de88bfa9b6e887c86025a88fdd
    GitTreeState: clean
    GoVersion: go1.20.12
    Compiler: gc
    Platform: linux/amd64
    FATA[0000] Argo CD server address unspecified

Adesso facciamo in modo di agganciare la CLI sul nostro ARGO-CD installato precedentemente sul cluster Kubernetes, niente di più semplice puntiamo la stessa url utilizzata per la web-gui, ci verranno chieste le credenziali admin e finito

    sudo argocd login argocd.marcofanuntza.it
    
    WARNING: server certificate had error: tls: failed to verify certificate: x509: certificate is valid for ingress.local, not argocd.marcofanuntza.it. Proceed insecurely (y/n)? y
    WARN[0008] Failed to invoke grpc call. Use flag --grpc-web in grpc calls. To avoid this warning message, use flag --grpc-web.
    Username: admin
    Password:
    'admin:login' logged in successfully
    Context 'argocd.marcofanuntza.it' updated

Con la CLI possiamo eseguire in maniera più diretta le stesse operazioni possibili dalla web-gui

    sudo argocd context
    CURRENT  NAME                     SERVER
    *        argocd.marcofanuntza.it  argocd.marcofanuntza.it

    sudo argocd cluster list --grpc-web
    SERVER                          NAME        VERSION  STATUS   MESSAGE                                                  PROJECT
    https://kubernetes.default.svc  in-cluster           Unknown  Cluster has no applications and is not being monitored.


Anche la CLI è stata installata, per avere una visione completa dei comandi possibili vi lascio il link della documentazione ufficiale [QUI](https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd/)



