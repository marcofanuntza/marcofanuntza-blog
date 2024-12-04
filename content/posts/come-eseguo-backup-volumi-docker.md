---
title: 'Come eseguo backup dei volumi Docker'
date: 2024-12-04T10:59:48+01:00
draft: false

categories:
 - Howto
 - Linux

tags:
 - Docker
 - Volumi
 - Raspberry
 - Ubuntu


cover:
  image: /img/gitlab-cover.webp
---

Come eseguo il backup dei volumi Docker? Niente di più semplice!

Tralasciando la pappardella su quanto sia importante avere dei backup vi mostro come eseguo il backup dei volumi Docker presenti sul mio Raspberry Pi 5

Eseguo applicazioni in self hosting e alcune sono dei container Docker, ho citato il Raspberry ma la stessa procedura può essere utilizzata su qualsiasi altra distribuzione Linux 

Partiamo dal presupposto che utilizziate Docker che esegue dei container e che questi abbiano dei volumi persistenti come nel mio caso.



**Ecco lo script**

Questo è lo script che utilizzo, ci sono i commenti per ogni fase e spero siano comprensibili per voi.

    #! /bin/sh
    PATH=/usr/local/bin:/usr/bin:

    #Script per eseguire backup dei volumi docker

    #Dichiaro Variabili
    DATE_BIN=$(command -v date)
    DATE=`${DATE_BIN} +%y-%m-%d--%H:%M:%S`

    DESTINATION=/home/marco/backup-docker-volume/backups/backup-docker/$DATE.tar.gz
    SOURCEFOLDER=/var/lib/docker/volumes/

    #Stop dei container in esecuzione, è importante per avere una situazione senza scritture sui volumi
    docker stop $(docker ps -q)

    #Creo un file zip tar.gz della directory dove Docker posiziona i volumi e salvo il file sulla directory di destinazione

    tar -cpzf $DESTINATION $SOURCEFOLDER

    #Restart dei container Docker precedentemente fermati
    docker restart $(docker ps -a -q)

    #Cancello i file backup più vecchi di 7 giorni
    find $DESTINATION/ -mtime +7 -type f -delete


Salviato il file si rende poi eseguibile, il nome che ho dato allo script è: script-bck.sh voi potete scegliere il nome che preferite basta che manteniate estensione del file .sh

    chmod a+x script-bck.sh


Sarebbe buona norma eseguirlo almeno una volta al giorno e considerando il fatto che i container devono essere momentaneamente fermati vi consiglio di eseguirlo la notte, ci pensa CRON (Obviously numero 1)

    0 0 * * * /usr/bin/sh /home/marco/backup-docker-volume/script-bck.sh

![Example image](/img/gitlab2.webp)


Il backup è importante ma la sua funzione potrebbe essere vana se lasciassimo i files sullo stesso host, muore questo perdiamo anche i backup.. (Obviously numero 2)

Per questo i files vengono direttamente scritti su uno share NFS presente sul mio server NAS (TruenasCore) Quello che infatti non vi ho specificato sullo scritp è che la variabile $DESTINATION è uno share NFS montato da fstab :P

