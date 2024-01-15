---
title: Il mio nuovo NAS con Truenas part II
author: Marco Fanuntza
date: 2024-01-13T08:53:54+00:00
url: /2024/01/13/il-mio-nuovo-nas-con-truenas-part2/

categories:
  - linux
  - server
tags:
  - Backup e Ripristino
  - Cluster
  - Cluster Domestico
  - Dischi
  - Gestione Server
  - NAS
  - Storage
  - Truenas
  - Virtualizzazione

cover:
    image: /img/truenas-logo-color.webp
---

Questo articolo continua il log partito da [QUI](https://marcofanuntza.it/2024/01/01/il-mio-nuovo-nas-con-truenas/) dove spiegavo la scelta e il perchè .

I componenti che attendevo sono arrivati, oltre la lista iniziale ho apportato alcune integrazioni aggiungendo una scheda PCI che di fatto mette a disposizione due porte SATA III aggiuntive. A questa scheda sono direttamente connessi i due dischi da 2,5 pollici, 2TB cadauno. Altra integrazione è un ulteriore banco di ram da 8GB.

La prima installazione è abbastanza semplice, se avete già avuto modo di installare una distro linux da pendrive USB sarà una passeggiata. Per creare la pendrive USB bootabile ho utilizzato il software **Balena Etcher** e la ISO di Truenas Core potete ovviamente trovarla sui repository ufficiali. La procedura guidata vi chiederà su quale disco installare l'OS, imposterete una password e successivamente sarà il turno della rete, finito! nel mio caso ha completato l'installazione in pochi minuti. A questo [link](https://www.truenas.com/blog/how-to-install-truenas-core/)  comunque potete seguire la pagina ufficiale con immagini passo per passo.

Ora potete anche scollegare monitor e tastiera dal server, tutto il resto potrà essere eseguito dalla web-gui di Truenas!


In questa galleria di immagini seguente potete vedere la dashboard iniziale che presenta con Truens CORE

{{< gallery match="images/*" sortOrder="asc" rowHeight="150" margins="15" thumbnailResizeOptions="600x600 q90 Lanczos" showExif=true previewType="blur" embedPreview=true loadJQuery=true >}}



.


**Prime operazioni** basilari che ho eseguito su TrueNAS **CORE**

  * **Impostato default gateway**: questo è indispensabile per cercare/installare aggiornamenti e installare plugin jail
  * **Impostato DNS**: ovviamente, no dns no party
  * **Creato primo pool**: primo passaggio per inizializzare lo storage a disposizione

**Le mie esigenze**


In aggiornamento&#8230;..
