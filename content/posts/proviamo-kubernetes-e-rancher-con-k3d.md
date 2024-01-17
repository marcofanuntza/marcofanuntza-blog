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

Kubernetes è un sistema di gestione (orchestratore) di container che è diventato di fatto lo standard per distribuire applicazioni containerizzate. È così popolare che molti sviluppatori professionisti non considerano alternative. 

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

**Procedura**


