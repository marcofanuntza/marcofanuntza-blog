---
title: Il mio nuovo NAS con Truenas
author: Marco Fanuntza
date: 2024-01-01T09:53:54+00:00
url: /2024/01/01/il-mio-nuovo-nas-con-truenas/
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
    image: img/nas-storage.webp

---

Acquistai il mio primo NAS nel lontano 2011, uno Zyxel NSA320 che nonostante tutto funziona ancora ma il peso degli anni si sente tutto, è ormai fuori supporto da tempo, nessuna opzione per upgrade o mod varie, versione tls obsoleta e opzioni su share NFS inesistenti. Quest&#8217;ultimo aspetto mi sta dando problemi con i backup dei container che ho su Proxmox e per questo ho preso la decisione&#8230; nuovo NAS sia!

Il primo aspetto preso in considerazione è: pappa pronta acquistando un prodotto commerciale dove basta inserire i dischi accendere e via, oppure costruirmelo da me?

Devo ammettere che i nuovi NAS in commercio hanno tutta una serie di features veramente interessanti, sono praticamente dei mini computer e oltre a salvare i dati permettono di installarci vari servizi, il famoso [Synology DS220+](https://amzn.to/3TL6UMe) ad esempio può essere un nas, un mediacenter, un reverse proxy, erogare virtual machines, erogare container docker, con questi ultimi due aspetti praticamente può fornire molteplici servizi, chi più ne ha più ne metta.


![Example image](/img/heading.webp)

Tutto questo però ovviamente ha un costo (alto a mio avviso) e altro aspetto da tenere in conto è che la maggior parte delle volte dovete aggiungerci i dischi, money! cantavano i Pink Floyd.

La mia curiosità batte la pigrizia della pappa pronta, quindi ho optato per costruirmelo da me, poi diciamocelo chiaro il bello è proprio quello! e un articolo descrivendo quanto fosse bello il Synology non l&#8217;avrei poi mai scritto.

La scelta è semplice, trovare un vecchio desktop pc, uno di quelli SFF (small form factor), su ebay e amazon ne trovate a bizzeffe ricondizionati, garantiscono prestazioni adeguate e consumi ridotti considerando che un NAS dovrebbe rimanere acceso H24.

Tra i tanti modelli, dopo varie analisi, la mia scelta è ricaduta su un Dell OPTIPLEX 3050 SFF, quello che ho trovato io ha una cpu non troppo vecchia Intel i3-7100, 8GB di ram DDR4 espandibile e aspetto scatenante la scelta è la presenza di uno slot M2 per storage nvme, dove ci andrà installato l&#8217;OS.

![Example image](/img/server-nas.webp)

Ok abbiamo la macchina, l&#8217;altra decisione da prendere è: che tipo di dischi utilizzare? le opzioni sono due, dischi super performanti SSD oppure classici e intramontabili HDD rotativi. Il prezzo dei dischi SSD è sceso tanto negli ultimi anni, ormai sono de facto i dischi da utilizzare su workstation e notebook, non sarebbero male da utilizzare anche sui NAS, c&#8217;è però un ma, i dischi SSD hanno una data di &#8220;scadenza&#8221; in base data dal numero di scritture eseguite, a questo si aggiunge un&#8217;aspetto che scoprirete dopo relativo al filesystem che verrà utilizzato ZFS, che per come funziona andrà sicuramente ad accorciarne ulteriormente la loro longevità.

Quindi si la mia scelta è stata&#8230; HDD rotativi! e saranno da 2,5 più facili da inserire dentro alla macchina SFF e meno energivori rispetto agli HDD classici da 3,5 pollici.

![Example image](/img/hdd-dischi.webp)

Infine arriva il software da utilizzare, scelta già fatta come avrete intuito dal titolo stesso dell&#8217;articolo, Truenas **CORE** sarà il software, il cuore e la mente del mio nuovo (vecchio) NAS.

![Example image](/img/truenas-logo-color.webp)

TrueNAS in versione **CORE** è una piattaforma di storage open-source basata su FreeBSD, progettata per fornire soluzioni di storage e condivisione di dati scalabili e affidabili. Originariamente conosciuta come FreeNAS, TrueNAS è stata ribattezzata per riflettere la sua evoluzione e le sue funzionalità avanzate. Ecco alcune delle sue caratteristiche principali:

  * **Storage Unificato:** Supporta protocolli come SMB/CIFS, NFS, AFP, iSCSI, S3 e altri, consentendo la centralizzazione della gestione dei dati e l&#8217;accesso da diversi sistemi operativi.
  * **ZFS File System:** Utilizza il file system ZFS, noto per la sua robustezza, gestione avanzata dei dati e funzionalità di snapshot e clone.
  * **Virtualizzazione e Containerization:** Fornisce funzionalità di virtualizzazione e containerization con supporto per VMware, Hyper-V, Bhyve e Docker.
  * **Ridondanza e Alta Disponibilità:** Offre opzioni avanzate per la ridondanza e l&#8217;alta disponibilità, garantendo la continuità degli accessi ai dati e proteggendo da guasti hardware.
  * **Sistema di Archiviazione Condiviso:** Configurabile come un sistema di archiviazione condiviso in reti aziendali, consentendo a diversi utenti di accedere e condividere dati in modo sicuro.
  * **Backup e Ripristino:** Include funzionalità di backup e ripristino, consentendo la creazione di snapshot e la pianificazione di backup automatici per garantire la sicurezza dei dati.
  * **Interfaccia Web Intuitiva:** Dispone di un&#8217;interfaccia web intuitiva che semplifica la gestione e la configurazione del sistema, rendendo accessibili molte funzionalità avanzate.

TrueNAS CORE e TrueNAS SCALE sono due varianti della piattaforma di storage TrueNAS, entrambe sviluppate da iXsystems. Ecco alcune differenze chiave tra i due:

**TrueNAS CORE:**

  * **File System ZFS:** TrueNAS CORE sfrutta il file system ZFS per avanzate funzionalità di gestione dati, snapshot e resistenza ai guasti.
  * **Stabilità e Affidabilità:** È noto per la sua stabilità e affidabilità, consigliato per utilizzi in ambienti critici.
  * **Orientato agli Utenti Esperti:** TrueNAS CORE è più indicato per utenti esperti e amministratori di sistema che desiderano maggiore controllo sulla configurazione.

**TrueNAS SCALE:**

  * **Basato su Debian:** TrueNAS SCALE ha una base Debian Linux, offrendo maggiore flessibilità nell&#8217;integrazione con software basato su Linux.
  * **Architettura più Moderna:** Progettato con un&#8217;architettura moderna e scalabile, adatto a carichi di lavoro su larga scala.
  * **Interfaccia Kubernetes:** TrueNAS SCALE presenta un&#8217;interfaccia Kubernetes nativa, semplificando l&#8217;implementazione e la gestione di applicazioni in contenitori.
  * **Approccio All-in-One:** Pensato come soluzione all-in-one con un modello di implementazione semplificato, adatto anche a utenti meno esperti.
  * **App Store:** TrueNAS SCALE include un App Store integrato con una varietà di applicazioni e servizi aggiuntivi facilmente installabili e gestibili.

In breve, mentre TrueNAS CORE offre stabilità consolidata e controllo avanzato, TrueNAS SCALE si orienta verso una flessibilità moderna, con facilità d&#8217;uso e integrazione di Kubernetes. La scelta tra i due dipende dalle esigenze specifiche dell&#8217;utente e dell&#8217;ambiente.

Io avendo già un homelab con Proxmox, non ho l&#8217;esigenza di avere un ulteriore ambiente che sarebbe stato una sorta di &#8220;doppione&#8221; con Truenas in versione SCALE e poi a me serve un NAS.

A questo punto dovrei mostrarvi il risultato&#8230;&#8230; si però dovrete attendere che mi arrivino i componenti!

Come diceva il grande JeeG Robot &#8220;Miwa lanciami i componenti!&#8221;

In aggiornamento&#8230;..

![Example image](/img/end-japan.webp)

[def]: /static/img/heading.webp