---
title: 'Installiamo Krew'
date: 2024-01-30T08:47:18+01:00
draft: false

tags:
 - kubernetes
 - kubectl
 - krew
 - howto
 - install
categories:
 - Kubernetes

cover:
  image: /img/krew-cover.jfif
---

Se siete arrivati a leggere questo articolo do per scontato conosciate già Kubectl, il non plus ultra della riga di comando per Kubernetes!

Sapevate che Kubectl può acquisire ancora più potenza grazie a Krew?

Krew è uno strumento che semplifica la gestione, l'installazione e l'aggiornamento di tutta una serie di plugin specifici per Kubectl, estendendone di fatto la già ampia funzionalità.

Il suo funzionamento può sembrare molto simile ai tradizionali gestori di pacchetti yum, apt, apk, brew e altri, utilizza un modello basato su repository per distribuire e gestire i plugin. L'installazione di nuovi plugin avviene in modo uniforme, fornendo una procedura standardizzata che semplifica il processo. 

Krew semplifica anche il processo di aggiornamento dei plugin, consentendo agli utenti di mantenere facilmente le loro estensioni aggiornate con le ultime funzionalità e correzioni di bug.

Tutto questo avviene direttamente dalla stessa interfaccia di comando Kubectl, gli utenti possano accedere a tutte le funzionalità aggiuntive avendo a disposizione un'ambiente totalmente integrato!

Se è la prima volta che sentite parlare di krew, posso che consigliarvi di dare un'occhiata alla documentazione ufficiale [QUI](https://krew.sigs.k8s.io/)

In questo articolo vi mostrerò come installarlo e proveremo alcuni plugin che ho avuto modo di testare.

**Prerequisiti**

- distribuzione linux
- git installato

**Procedimento**
Basandoci sulla documentazione ufficiale utilizzeremo uno script che si occuperà di installare Krew in autonomia, lo script è il seguente:

    (
      set -x; cd "$(mktemp -d)" &&
      OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
      ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
      KREW="krew-${OS}_${ARCH}" &&
      curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
      tar zxvf "${KREW}.tar.gz" &&
      ./"${KREW}" install krew
    )

dopo che lo script avrà terminato editiamo il file .bashrc inserendo nell'ultima riga l'export path che segue

    export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"

salvate il file e ricaricate la vostra sessione shell, il comando che segue se tutto è andato bene vi restituirà lo stesso output

    kubectl krew version
    OPTION            VALUE
    GitTag            v0.4.4
    GitCommit         343e657
    IndexURI          https://github.com/kubernetes-sigs/krew-index.git
    BasePath          /root/.krew
    IndexPath         /root/.krew/index/default
    InstallPath       /root/.krew/store
    BinPath           /root/.krew/bin
    DetectedPlatform  linux/amd64


Ok a questo punto abbiamo Krew installato, possiamo provare qualche plugin. Testiamo ad'esempio il plugin **Tree**, come potete vedere per installare il plugin basterà un semplice comando

    kubectl krew install tree

La funzione principale del plugin **Tree** è fornire una rappresentazione visuale ad albero delle risorse all'interno di un cluster Kubernetes. Invece di visualizzare le risorse in un formato tabellare o di elenco, il plugin kubectl tree organizza gerarchicamente le risorse in una struttura ad albero, rendendo più facile comprendere la relazione tra di esse.

Ecco un'esempio con il deployment di Velero

     kubectl tree deployment velero -n velero
     NAMESPACE  NAME                             READY  REASON  AGE
     velero     Deployment/velero                -              2y221d
     velero     ├─ReplicaSet/velero-668d7f99cc   -              630d
     velero     │ └─Pod/velero-668d7f99cc-cqbpf  True           287d
     velero     ├─ReplicaSet/velero-86c77779f6   -              630d
     velero     └─ReplicaSet/velero-8cb65ddbc    -              2y221d
    
Proviamo poi il plugin **Clog** che banalmente colora l'output dei logs quando visualizzati da kubectl

    kubectl krew install clog

Proviamo il plugin **Outdated** che come potete immaginare farà una scansione delle immagini utilizzate segnalando tag cottente in utilizzo e versione latest, aiutandoci a individuare quali sarebbe meglio aggiornare...

    kubectl krew install outdated

    kubectl outdated

![Example image](/img/Cattura.png#center)

Come già scritto, sono presenti una mirade di plugin, al momento ne contiamo 231, vi rimando alla pagina [ufficiale](https://krew.sigs.k8s.io/plugins/) e scegliete il plugin che desiderate!


![Example image](/img/Cattura1.png#center)
    

    
