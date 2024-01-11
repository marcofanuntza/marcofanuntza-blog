---
title: Ridimensionare disco container su Proxmox
author: Marco Fanuntza
date: 2023-12-17T09:10:01+00:00
url: /2023/12/17/ridimensionare-disco-container-su-proxmox/

categories:
  - linux
  - server
tags:
  - Backup e Ripristino
  - Cluster
  - Containerizzazione
  - Gestione Server
  - Linux Containers
  - LVM
  - Proxmox
  - Ridimensionare
  - Virtualizzazione

cover:
    image: /img/dischi.webp
---

Spesso può capitare la necessità di dover ridimensionare il disco di un container creato troppo grande o relativamente piccolo, a dire la verità spesso si tende ad aumentare lo spazio disco, vuoi crescita dei logs, aggiunta di nuove funzioni, in questo caso specifico però mostreremo come ridurre lo spazio.

Per chi come me utilizza Proxmox in ambito domestico dovrà fare subito i conti con la mancanza di una LAN a 10GbE, questo si tradurrà in migrazioni tra i nodi eccessivamente lunghe, la regola quindi deve essere: container piccolo = veloce. Quando si crea un container può succedere di non aver ben chiaro in mente quanto dimensionare lo spazio disco, ma niente paura si ridimensiona!


![Example image](/img/lvm-image.webp#center)


Proxmox di default quando crea il container utilizza LVM che sta per (in lingua inglese logical volume manager) in italiano gestore logico dei volumi, ringraziamo quindi questa scelta perché ci permetterà di ridimensionare i nostri container con dei semplici passaggi che scopo di questo articolo vi elencherò qui di seguito.

Prima di tutto come sempre vi consiglio di eseguire un backup del container interessato, terminato il backup saremo pronti a partire.

Per i comandi che seguono sarà necessario operare da >_Shell, la potete attivare da Proxmox e sarà sul nodo che in quel momento sta eseguendo il container interessato. 
Io personalmente preferisco operare direttamente da shell ssh ma sarà uguale.

Iniziamo con individuare il volume utilizzato dal container eseguendo questo comando:

    lvdisplay | grep "LV Path\|LV Size"

il risultato sarà simile al seguente; simile non uguale sia ben chiaro 😉

    LV Size                141.23 GiB
    LV Path                /dev/pve/swap
    LV Size                8.00 GiB
    LV Path                /dev/pve/root
    LV Size                69.37 GiB
    LV Path                /dev/pve/vm-101-disk-0
    LV Size                22.00 GiB

dovrete prestare attenzione alle ultime due righe, queste infatti si riferiscono al disco (volume) del container che andremo a modificare, che potete intuire sarà vm-101-disk-0 sul percorso /dev/pve/vm-101-disk-0

intanto procediamo con spegnere il container, si scusate non lo avevo ancora scritto, queste operazioni vanno eseguite con il container spento, eseguite il comando seguente per spegnere il container

    pct stop 101

adesso partiamo con ridimensionare il filesystem del volume, nel nostro caso intento è di farlo diventare da 10GB quindi eseguiamo:

    resize2fs /dev/pve/vm-101-disk-0 10G

il risultato ci restituirà:

    resize2fs 1.47.0 (5-Feb-2023)
    Resizing the filesystem on /dev/pve/vm-101-disk-0 to 2621440 (4k) blocks.
    The filesystem on /dev/pve/vm-101-disk-0 is now 2621440 (4k) blocks long.


a questo punto siamo pronti con il comando che effettivamente ridimensionerà il nostro volume, il comando **lvreduce** fa parte della suite messa a disposizione da LVM, per i più curiosi c&#8217;è sempre il MAN di linux per approfondimenti, che detto in Sardo &#8220;mai ammanchidi&#8221; (mai manchi)

    lvreduce -L 10G /dev/pve/vm-101-disk-0
    WARNING: Reducing active logical volume to 10.00 GiB.
    THIS MAY DESTROY YOUR DATA (filesystem etc.)
    Do you really want to reduce pve/vm-101-disk-0? y/n]: y

confermate con &#8220;y&#8221; senza paura! avevate fatto il backup si? Il risultato che restituirà il comando sarà il seguente:

    Size of logical volume pve/vm-101-disk-0 changed from 22.00 GiB (5632 extents) to 10.00 GiB (2560 extents).
    Logical volume pve/vm-101-disk-0 successfully resized.

le operazioni a questo punto sono praticamente completate, ma manca un ultimo piccolo passaggio e sarà relativo alla modifica della configurazione del container, dobbiamo semplicemente fargli sapere che le dimensioni del disco sono cambiate

    vi /etc/pve/lxc/101.conf

la stringa rootfs dovrà cambiare semplicemente da 22GB a 10GB nel punto &#8220;size&#8221;

    rootfs: local-lvm:vm-101-disk-0,size=10G

ora potete finalmente accendere il container e per avere la certezza della buona riuscita procedete con accedere nel vostro container su console e verificate lo spazio disco con un semplice df -h

![Example image](/img/Image-proxmox-1.webp#center)

Questo è tutto, il ridimensionamento è possibile anche in caso contrario, quindi aggiungendo spazio disco, il comando in quel caso sarà lvextend +10G e resize2fs ma comunque sia Proxmox in quel caso vi viene in aiuto permettendovi di eseguirlo direttamente da gui e completamente automizzato.