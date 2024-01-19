---
title: 'Automazione con Ansible'
date: 2024-01-18T15:01:35+01:00
draft: true

categories:
 - linux
 - server
tags:
 - ansible
 - automation
 - playbook

cover:
  image:

---

Automatic for the people è un album dei R.E.M. mi è venuto in mente quando ho pensato che Ansible è un prodotto di "automation" IT.

Ansible è una potente e flessibile piattaforma di automazione IT progettata per semplificare e automatizzare una vasta gamma di compiti, processi e operazioni legate all'infrastruttura e allo sviluppo del software.

Di seguito alcuni aspetti salienti:

- Ansible si distingue per la sua facilità d'uso, grazie a una sintassi dichiarativa basata su YAML è accessibile anche a chi ha una conoscenza limitata della programmazione.

- Ansible opera tramite connessioni SSH, eliminando la necessità di installare agenti o client aggiuntivi sulle macchine di destinazione.

- Ansible consente di definire la configurazione di un'intera infrastruttura (Infrastracture as a Code) attraverso un semplice file di testo, migliorandone riproducibilità e versioning.

- Oltre alla gestione delle configurazioni può automatizzare una vasta gamma di operazioni inclusi il rilascio di applicazioni, il monitoraggio e la scalabilità.

- Ansible è progettato per funzionare su diverse piattaforme e sul cloud, offrendo flessibilità e portabilità.

- Ansible inoltre è in grado di gestire ambienti di varie dimensioni, rendendolo adatto per piccoli progetti fino a progetti più articolati e complessi.


Nel mio ambito lavorativo Ansible era da un pò che ne sentivo parlare ma personalmente non avevo mai avuto modo di utilizzarlo, con questo post sul mio blog unisco l'utile al dilettevole cogliendo l'occasione per conscerlo meglio!

Partiamo dalla basi prima di tutto installandolo, eseguirò i test su una VM con distribuzione Ubuntu 22-04, ansible e i relativi moduli python verranno installati tramite APT
    
    #Installo ansible
    sudo add-apt-repository --yes --update ppa:ansible/ansible
    sudo apt install ansible-core

    #Verifico la versione installata
    sudo ansible --version
    
    ansible [core 2.15.8]
      config file = /etc/ansible/ansible.cfg
      configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
      ansible python module location = /usr/lib/python3/dist-packages/ansible
      ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
      executable location = /usr/bin/ansible
      python version = 3.10.12 (main, Nov 20 2023, 15:14:05) [GCC 11.4.0] (/usr/bin/python3)
      jinja version = 3.0.3
      libyaml = True

Adesso che Ansible è installato dobbiamo avere a disposizione dei server o delle VM con cui utilizzarlo, per fare questo io personalmente ho creato 3 istanze LXC sul mio cluster Proxmox, anche in questo caso la distribuzione utilizzata è ubuntu, giusto a titolo d'esempio, qualsiasi altra distribuzione linux andrebbe bene.

Questo è il recap delle macchine che ho a disposizione:

ans-serv-01 192.168.1.149
ans-serv-02 192.168.1.150
ans-serv-03 192.168.1.154

Ora i punti cardine principali di Ansible sono i file di configurazione "inventario" e "playbook" 

Il file inventario di Ansible contiene informazioni su tutti gli host che Ansible andrà a gestire, in questo file si ha la possibilità di organizzare gli host in diversi gruppi in base ai loro ruoli o funzioni, come ad esempio web-server, database, server di frontend, oppure possiamo categorizzarli in base al sistema operativo.
Il file in questione lo troviamo nel path /etc/ansible ed'è chiamato hosts, andando a editarlo scoprirete che in parte è già strutturato per aiutarne la comprensione, basterà quindi eliminare i commenti e adattarlo a piacimento.



Noi in questo esempio andiamo a editarlo così per questa sola parte

    [webservers]
    ## alpha.example.org
    ## beta.example.org
    192.168.1.149
    192.168.1.150
    192.168.1.154

come potete intuire sono gli IP delle macchine elencate in precedenza, le ho inserite nel gruppo "webserver"





    

    


    
