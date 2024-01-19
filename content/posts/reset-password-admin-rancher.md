---
title: 'Reset password utente admin su Rancher'
date: 2024-01-19T08:57:35+01:00
draft: false

categories:
 - linux
 - server
 - rancher

tags:
 - rancher
 - password
 - reset
 - admin
 - kubectl
 - kubernetes

cover:
   image: img/passwd-admin-rancher.webp

---

**Rancher come eseguire un reset della password dell'utente Admin**

Si può capitare a tutti di dimenticare una password, ad alcuni spesso, ma niente paura possiamo eseguire un reset eseguendo questi semplici comandi che seguono..

L'esempio che segue mostra come eseguire il reset della password dell'utente admin di Rancher installato all'interno di un cluster Kubernetes, le operazioni in parte sono valide anche nel caso il vostro Rancher fosse stato installato su un semplice container docker.


**Prerequisiti**

- avere accesso al cluster Kubernetes tramite kubectl
- avere accesso al server che esegue docker (nel secondo caso)


**Procedura**

Tramite kubectl individuiamo il pod che sta erogando il servizio Rancher

    kubectl get pods -n cattle-system | grep Running

    rancher-webhook-7476c74c6c-scbvs   1/1     Running                  0             42m
    rancher-7484b7b4c5-jb9dt           1/1     Running                  0             42m

a questo punto dobbiamo entrare all'interno della console del pod rancher-7484b7b4c5-jb9dt, per farlo eseguiremo

    kubectl exec --stdin --tty rancher-7484b7b4c5-jb9dt -n cattle-system -- /bin/bash

noterete che il prompt dei comandi sarà cambiato, siamo all'interno del pod e possiamo eseguire dei comandi diretti, quello specifico che farà al caso nostro è un semplice "reset password"

     reset-password
     New password for default admin user (user-m258d):
     tQXngoMxYBtugFp4Xcjg

dopo aver copiato la password potete uscire dalla console con "exit". 

Con questi semplici passaggi la vostra password admin è stata rigenereata e sarete nuovamente in grado di accedere al vostro Rancher con utenza admin!

