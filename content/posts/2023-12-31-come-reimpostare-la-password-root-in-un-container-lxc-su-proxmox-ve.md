---
title: Come reimpostare la password root in un container LXC su Proxmox VE
author: Marco Fanuntza
date: 2023-12-31T17:34:21+00:00
url: /2023/12/31/come-reimpostare-la-password-root-in-un-container-lxc-su-proxmox-ve/


categories:
  - linux
  - server
tags:
  - Backup e Ripristino
  - Cluster
  - Cluster Domestico
  - dimenticata
  - Gestione Server
  - Linux Containers
  - LXC
  - password
  - Proxmox
  - reset
  - Root
  - Virtualizzazione

cover:
    image: img/pinguino.webp

---


**Problema** 

Abbiamo dimenticato la password di un container LXC in esecuzione su Proxmox e non abbiamo alternative se non quella di resettarla.

**Soluzione**

  *Effettuare l&#8217;accesso sulla web GUI del vostro cluster Proxmox.


  *Individuate il container LXC per il quale si desidera reimpostare la password e ricordarsi l&#8217;ID del container. Ad esempio, se vediamo un container chiamato:

    105 (passwdDimenticata) 

  *105 saràil suo ID, passwdDimenticata sarà il suo nome.

  *Avviare il container nel caso non lo fosse già

  *Ora connettersi via SSH sull&#8217;host di Proxmox che sta eseguendo il container (come utente root) o aprite una Shell/Console sulla web GUI di Proxmox, sempre sul nodo che sta eseguendo il container

  *Utilizzare il seguente comando per collegare la sessione al container LXC

    lxc-attach -n 105




  *A questo punto ci troveremo all&#8217;interno della shell del container e potremo eseguire i comandi, nello specifico per cambiare la password di root 

    passwd

  *Digitare la nuova password, premere il tasto Invio, quindi digitare nuovamente la password e premere nuovamente Invio per confermare la nuova password di root del container.
-Un
a volta completato, è possibile effettuare l&#8217;accesso al container con la nuova password.
