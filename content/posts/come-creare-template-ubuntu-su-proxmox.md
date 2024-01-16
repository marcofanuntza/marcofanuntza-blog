---
title: 'Come creare template Ubuntu su Proxmox'
date: 2024-01-16T09:33:00+01:00
draft: false

tags:
 - linux
 - ubuntu
 - proxmox
 - template

catergories:
 - linux
 - proxmox
 - server

cover:
    image: /img/template-ubuntu-px.webp
 
---

**Premessa**

Questa guida mostra i comandi da eseguire per creare un template di una VM da utilizzare su Proxmox, la distro utilizzata è Ubuntu e l'immagine sarà una versione specifica per il cloud.

Le Immagini Cloud sono piccole immagini certificate e pronte per il cloud, hanno Cloud Init preinstallato e pronto per accettare una Cloud Config.

I comandi verranno tutti eseguiti da shell all'interno di un nodo Proxmox

**Procedura**

Iniziamo scaricando l'immagine Ubuntu dalla pagina specifica [Ubuntu Cloud Images](https://cloud-images.ubuntu.com/) per questa guida utilizzeremo Ubuntu Server 24.04 LTS (Noble Numbat)

    wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img

Creiamo la VM che diventerà la base per il successivo template

    qm create 2000 --memory 2048 --core 2 --name ubuntu-cloud-noble --net0 virtio,bridge=vmbr0

Importiamo l'immagine precedentemente scaricata sul volume locale mettendola a disposizione della VM

    qm importdisk 2000  noble-server-cloudimg-amd64.img local-lvm

Attendiamo il trasferimento e poi procederemo con collegare il nuovo disco alla VM come un'unità SCSI sul controller SCSI

    qm set 2000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-2000-disk-0

Adesso aggiungiamo il cloud-init drive, questo è una parte indispensabile per la creazione del template, permette di cambiare la configurazione dell'immagine ogni volta che verrà utilizzato il template

    qm set 2000 --ide2 local-lvm:cloudinit

Rendiamo l'unità cloud-init avviabile e limitiamo i permessi al solo BIOS

    qm set 2000 --boot c --bootdisk scsi0

Aggiungiamo infine serial console

    qm set 2000 --serial0 socket --vga serial0


**ATTENZIONE** a questo punto non avviate la VM appena creata, siamo pronti per creare il template eseguendo quest'ultimo semplice comando

    qm template 2000

    Renamed "vm-2000-disk-0" to "base-2000-disk-0" in volume group "pve"
    Logical volume pve/base-2000-disk-0 changed.
    WARNING: Combining activation change with other commands is not advised.

Adesso avete a disposizione un template che vi permetterà di generare N immagini Ubuntu senza il bisogno di dover ogni volta rieseguire una nuova installazione, **insomma è un template!**

Chiudiamo la guida con un semplice esempio creando una nuova VM partendo dal template appena creato

    qm clone 2000 110 --name ubuntu-noble-01 --full

Verificando possiamo notare la VM appena creata con ID 110 e nome ubuntu-noble-01

    qm list
    VMID NAME                 STATUS     MEM(MB)    BOOTDISK(GB) PID       
    103 haos11.2             stopped    4096              32.00 0         
    110 ubuntu-noble-01      stopped    2048               3.50 0         
    2000 ubuntu-cloud-noble   stopped    2048               3.50 0 

Noterete anche che l'immagine ha un disco di soli 3.5GB, nessun problema in base alle vostre esigenze potete ridimensionare il disco eseguendo questo successivo comando

    qm resize 110 resize scsi0 15G
    
    Size of logical volume pve/vm-110-disk-0 changed from 3.50 GiB (896 extents) to 15.00 GiB (3840 extents).
    Logical volume pve/vm-110-disk-0 successfully resized.


Adesso siamo pronti per far partire la nostra nuova VM creata dal template.. ma manca qualcosa, spero i più attenti ci abbiano fatto caso, che user e passwd dobbiamo utilizzare per il login?

E' qui che entra in gioco cloud-init, aprite la web-gui di Proxmox e posizionatevi sulle voci di menù relative alla VM, cliccate sulla voce **Cloud-init** e noterete le voci che si possono editare, modificatele e provate a far partire la VM. Vi consiglio comunque di utilizzare direttamente la vostra chiave SSH

![Example image](/img/template-ubuntu-px1.webp)


La guida termina qui, se velete utilizzare una distribuzione diversa da Ubuntu niente vi vieta di farlo, la guida va semplicemente adattata utilizzando un'immagine diversa.









