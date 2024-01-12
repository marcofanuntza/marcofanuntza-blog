---
title: Ho scelto Hugo!
author: Marco Fanuntza
date: 2024-01-12T10:18:27+00:00
url: /2024/01/12/ho-scelto-hugo/
draft: false

categories:
  - linux
  - server
  - cms
tags:
  - Hugo
  - GO
  - Blog
  - PaperMod
  - Linux
  - CMS

cover:
    image: /img/hugo-icon2.webp
---


**Introduzione:**

Quando ho deciso di riscrivere sul blog il cms scelto inizialmente era stato Wordpress, per ambito lavorativo avevo già gestito server wordpress decine di volte, avevo avuto anche un'esperienza come writer assiduo sul defunto blog actioncamitalia, la scelta quindi si era basata esclusivamente sull'esperienza passata.
Dopo un pò mi sono accorto però che per le mie esigenze, per le esigenze di questo blog specifico, l'utilizzo delle risorse necessarie per worpress erano sprecate, insomma non ne avevo bisogno. 

Da qui è partita la ricerca su un nuovo CMS per il mio blog.

Ci sono molte piattaforme di blogging. Essendo un sistemista, le mie esigenze per una piattaforma di blogging potrebbero differire da quelle della maggior parte dei blogger. 
Io vorrei che il mio blog sia:

- Facile da mantenere per quanto riguarda gli aggiornamenti del software
- Semplice nelle sue funzionalità
- Facile per me da configurare
- Trasparente su ciò che sta accadendo sotto il cofano


**Che cos'è Hugo?**

Hugo è un framework open source per la generazione di siti web statici. Creato utilizzando il linguaggio di programmazione Go (o Golang), Hugo si distingue per la sua velocità straordinaria nella generazione di contenuti statici. Alcune caratteristiche chiave di Hugo includono:

**Velocità**: Grazie alla sua implementazione in Go, Hugo è notevolmente veloce nella generazione di siti web statici, rendendo il processo efficiente e immediato.

**Semplicità**: Hugo è progettato per essere facile da usare e comprendere. La sua struttura chiara e la documentazione completa facilitano la creazione e la gestione di siti web.

**Flessibilità**: Supporta temi e layout personalizzabili, consentendo agli sviluppatori di adattare l'aspetto e la struttura del sito secondo le proprie esigenze.

**Generazione di Contenuti Statici**: Hugo crea siti web completamente statici, eliminando la necessità di un server di backend. Ciò li rende sicuri, facili da distribuire e veloci da caricare.


**Installare Hugo**

Su qualsiasi distribuzione linux è abbastanza banale installare Hugo, i vari snap, apt e yum hanno nei loro repository i pacchetti necessari, ma c'è da dire che spesso non sono aggiornati. Il mio consiglio è scaricarvi il pacchetto più recente e installarlo a manina.

Nel mio caso specifico ho deciso di installare Hugo su un container LXC erogato dal mio cluster Proxmox, distribuzione ho scelto una ubuntu 23-10, dal sito di Hugo ho scaricato l'ultima release disponibile del pacchetto deb


    wget https://github.com/gohugoio/hugo/releases/download/v0.121.2/hugo_0.121.2_linux-amd64.deb

eseguito il comando di installazione:

    dpkg -i hugo_0.121.2_linux-amd64.deb

qui il risultato:

    hugo version
    hugo v0.121.2-6d5b44305eaa9d0a157946492a6f319da38de154 linux/amd64 BuildDate=2024-01-05T12:21:15Z VendorInfo=gohugoio

**Primi Passi**

Prima di tutto vi consiglio di leggere la documentazione di Hugo e consiglio di partire dalla [quick-start](https://gohugo.io/getting-started/quick-start/)

Io non l'ho seguita alla lettera ma ho apportato alcune modifiche, procediamo con la creazione del nostro primo sitoweb con Hugo, posizionatevi in un path qualunque sul vostro server e eseguite il seguente comando:

    hugo new site mio-nuovo-blog --format yaml

descrivendo nello specifico il comando, la prima parte istruisce hugo per creare un sito, mio-nuovo-blog sarà il nome del sito e l'opzione -f yml farà in modo che i file di configurazione vengano formattati in formato yaml invece che toml, a mio avviso più semplice da capire a prima vista.

Scoprirete che hugo ha creato una nuova directory con il nome del sito e posizionato al suo interno tutti i files necessari, dovreste ritrovarvi in questa simile situazione

![Example image](/img/hugo-articolo1.webp#center)
 
Il passo successivo sarà scegliere e abiltare un tema da utilizzare, date un'occhiata sul sito ufficiale e scegliete quello che preferite. Sono quasi tutti ben documentati e la procedura per attivarli è quasi sempre la stessa, sostanzialmente va utilizzato git e scaricato il repository all'interno del path creato in precedenza da Hugo.

Io ho scelto il tema **PaperMod** e l'ho abilitato come submodulo di git con i seguenti comandi:

    git init

questo vi servirà anche nel caso voi decidiate di versionare il vostro sito su GitHub, procediamo poi con:

    git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
    git submodule update --init --recursive # needed when you reclone your repo (submodules may not get cloned automatically)

Ora troverete il tema installato e a disposizione per il vostro nuovo sito, i files del tema sono posizionati all'interno del path themes/PaperMod.

Adesso verrà il bello per voi, il file che editerete principalmente sarà config.yaml, questo è di fatto il punto principale di controllo per il vostro sito Hugo.

Ma finiamo con un'ultimo comando poi lascerò a voi il bello di configurarvi il vostro sito Hugo!

**Come creo il mio primo post?**

Bene niente di più semplice, per farlo utilizziamo nuovamente un comando hugo

    hugo new content posts/il-mio-primo-post.md

questo comando creerà un nuovo post vuoto e come potrete intuire sarà sul path content/posts/


A questo punto credo sia arrivato il momento di lasciarvi scoprire come andare avanti con Hugo in base alle vostre esigenze, se volete comunque dare un'occhiata al codice del mio blog, questo stesso che sta leggendo, vi lascio il link al mio repository su [GitHub](https://github.com/marcofanuntza/marcofanuntza-blog)

**Peace & Love!**
