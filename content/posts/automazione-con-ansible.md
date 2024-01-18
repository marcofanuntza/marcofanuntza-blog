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

Partiamo dalla basi prima di tutto installandolo, eseguirò i test su un container LXC con distribuzione Ubuntu, ansible e i relativi moduli python considerando questo un'ambiente di test verranno installati tramite APT, se non abbiamo i pacchetti aggiornatissimi non sarà un problema.
    
    #Installo ansible
    sudo apt install ansible

    #Verifico la versione installata
    sudo ansible --version
    ansible [core 2.14.9]
    config file = None
    configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
    ansible python module location = /usr/lib/python3/dist-packages/ansible
    ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
    executable location = /usr/bin/ansible
    python version = 3.11.6 (main, Oct  8 2023, 05:06:43) [GCC 13.2.0] (/usr/bin/python3)
    jinja version = 3.1.2
    libyaml = True


    

    


    
