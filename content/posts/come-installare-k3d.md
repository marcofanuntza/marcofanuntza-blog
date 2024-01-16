---
title: 'Come installare K3D'
date: 2024-01-16T09:52:46+01:00
draft: true

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

K3D è un "wrapper" che come scrive [Wikipedia](https://it.wikipedia.org/wiki/Wrapper) "è un'avvolgitore, un modulo software che ne "riveste" un altro" 

Si la traduzione dall'inglese non è felicissima, in questo caso specifico consente di eseguire K3S, che è la distribuzione minimale di Kubernetes sviluppata da Rancher Labs, all'interno di Docker. 

In altre parole, K3D semplifica la creazione e la gestione di cluster Kubernetes leggeri e portatili che utilizzano K3S, rendendo il processo più agevole, specialmente per lo sviluppo locale su Kubernetes.

È importante notare che K3D è un progetto guidato dalla community, non è un prodotto ufficiale di Rancher (SUSE) che ha creato e mantiene K3S. Tuttavia, K3D offre un'opzione comoda e flessibile per coloro che desiderano utilizzare K3S all'interno di container Docker.
