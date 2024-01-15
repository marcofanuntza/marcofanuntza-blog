---
title: 'Come installare Docker e Docker Compose su Ubuntu'
date: 2024-01-15T13:34:41+01:00
draft: false


cover:
---

Come installare Docker e Docker compose su Ubuntu

Questa guida elenca passo per passo la procedura da seguire per installare docker, docker compose e containerd su distribuzione Ubuntu.

**Prerequisiti:**

- server o workstation con distribuzione ubuntu
- accesso alla rete per scaricare i pacchetti


**Procedura**

Tutti i comandi verranno eseguiti da terminale, e in precedenza avevate già provato un'installazione di Docker sarebbe opportuno rimuoverla eseguendo il comando che segue:

    sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras

adesso si può procedere con l'installazione, si parte prima di tutto aggiungendo il repository ufficiale Docker

    # Aggiungiamo la chiave **GPG key** ufficiale del repository Docker:
    sudo apt-get update
    sudo apt-get install ca-certificates curl gnupg
    sudo install -m 0755 -d /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    sudo chmod a+r /etc/apt/keyrings/docker.gpg

aggiungiamo il repository ufficiale Docker al sistema APT:
    
    echo \
    "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update


qui il comando per installazione dei pacchetti necessari:

    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

verifichiamo le versioni installate di Docker e Docker compose:

    docker -v
    docker compose

a questo punto possiamo considerare completata l'installazione, in via opzionale ci resta solamente abilitare il nostro user per utilizzo del comando docker senza il bisogno di utilizzare ogni volta sudo, per farlo aggiungiamo semplicemente lo user al gruppo docker

    sudo usermod -aG docker $USER

that'all folks!
