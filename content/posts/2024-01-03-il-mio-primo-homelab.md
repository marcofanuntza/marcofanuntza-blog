---
title: Il mio primo HomeLab
author: Marco Fanuntza
date: 2023-11-30T09:10:01+00:00
draft: true
url: /?p=458
categories:
  - linux
  - server
tags:
  - Backup e Ripristino
  - Cluster
  - Cluster Domestico
  - Containerizzazione
  - Gestione Server
  - Linux Containers
  - LVM
  - Proxmox
  - Virtualizzazione

---
**Fase 1. La pianificazione**


Consideriamo queste premesse, sarà un home cluster con risorse computazionali non esagerate e lo stesso dovrà restare acceso H24, la scelta quindi ricadrà su hardware che permetterà consumi molto bassi in idle.

Inizialmente avevo valutato delle single board ARM e X86, nello specifico la ZimaBoard che tanti youtuber sponsorizzano, ma visto il prezzo e visti i limiti ho optato per dei mini pc usati che si trovano a prezzi relativamente bassi su ebay e che offrono prestazioni di tutto rispetto con ampie opzioni di upgrade possibili rispetto alle single board.



**Fase 2. La lista della spesa:**

HARDWARE

1 HP EliteDesk 705 G4 16GB Ram NVEM 256GB CPU AMD Ryzen 5 Pro 2400G (Quadcore con 8 tread)  
1 Lenovo Thinkcentre M715Q 32GB Ram NVEM 256GB CPU AMD Ryzen 5 Pro 2400GE (Quadcore con 8 tread)

1 Raspberry pi3 per il quorum e per ospitare nginx proxy manager

1 NAS Zixel NSA 320 per backup e con share NFS

Il raspberry era già in mio possesso, verrà utilizzato per il quorum del cluster Proxmox e per ospitare nginx proxy manager, le altre due macchine sono state acquistate usate, sono dei prodotti ricondizionati, la cpu è stata scelta esplicitamente, purtroppo solo una ha la GE un TPW di 35W ma comunque in idle consumano pochissimo entrambe.

La scelta dello storage è stata combattuta, Proxmox mette a disposizione alcune variabili in fase di installazione, classico EXT4 con LVM, ZFS, BTRFS, considerando che i server hanno un solo disco a bordo, non ho bisogno di importante spazio disco, posso fare a meno di un RAID, ZFS è si tanto figo e mi incuriosisce parecchio ma alla fine ho optato per il caro e semplice EXT4 con LVM, per le mie necessità è la scelta più logica e semplice.  
Per il backup ho a disposizione il NAS con TrueNAS che ho descritto <a href="https://marcofanuntza.it/2024/01/01/il-mio-nuovo-nas-con-truenas/" target="_blank" rel="noopener" title="">in questo articolo</a>. In questo caso posso optare per l&#8217;adozione del filesystem ZFS



SOFTWARE

Proxmox VE, hypervisor di tipo 1, permette virtualizzazione VM e container… e tanto altro.

Dominio già acquistato su ionos, offre gratuitamente casella email

Cloudflare, servizio dns, ddos protection, certificato ssl, tutto gratuitamente scegliendo il piano free per &#8220;hobbysti&#8221;

Nginx Proxy Manager che fa da reverse proxy e unico punto pubblicato esternamente su https/443

WordPress su container per erogare questo stesso blog che state leggendo