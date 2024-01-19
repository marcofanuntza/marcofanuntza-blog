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

![Example image](/img/ansible-esempio1.webp#center)

Noi in questo esempio andiamo a editarlo così per questa sola parte

    [webservers]
    ## alpha.example.org
    ## beta.example.org
    192.168.1.149
    192.168.1.150
    192.168.1.154

come potete intuire sono gli IP delle macchine elencate in precedenza, le ho inserite nel gruppo "webserver". Verifichiamo con il comando seguente

    sudo ansible-inventory --list -y
    all:
      children:
        webservers:
          hosts:
          192.168.1.149: {}
          192.168.1.150: {}
          192.168.1.154: {}

Come annunciato in precedenza Ansible non utilizza client o agent all'interno dei server da gestire ma si avvale della sola connessione SSH, per utilizzarla però deve essere in grado di connettersi sui server di destinazione, per fare questo è necessario copiare la chiave SSH dal server Ansible verso i server da gestire. Questo inoltre eviterà di dover utilizzare la passwd per connettersi che andrebbe a inficiare sull'automatismo non rendendolo possibile

Iniziamo con creare la chiave sul server Ansible

    #Date sempre invio dopo il comando
    sudo ssh-keygen
    
    Generating public/private rsa key pair.
    Enter file in which to save the key (/root/.ssh/id_rsa):
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in /root/.ssh/id_rsa
    Your public key has been saved in /root/.ssh/id_rsa.pub
    The key fingerprint is:
    SHA256:ZRAsk+qjBjQjpsLdEF39qdUmOa5ARx7QPFr6EEQhbDY root@ubuntu-22-04-lts
    The key's randomart image is:
    +---[RSA 3072]----+
    |    o.+OO.       |
    |   . E=..X       |
    |    +..oB * +    |
    |.= ..  = = B o   |
    |* +.o . S + +    |
    |+. .o. . o .     |
    |.. . .  . .      |
    |  o      .       |
    | .               |
    +----[SHA256]-----+


la chiave pubblica adesso va copiata sui server da gestire, il comando che segue va eseguito su tutti e tre i server, la password andrà inserita la prima volta.

    sudo ssh-copy-id root@192.168.1.149
    sudo ssh-copy-id root@192.168.1.150
    sudo ssh-copy-id root@192.168.1.154


Verifichiamo il buon esito eseguendo questo comando

    sudo ansible all -m ping -u root
    192.168.1.149 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
    }
    192.168.1.150 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
    }
    192.168.1.154 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
    }

**Adesso Ansible è pronto!**

Dopo il file inventario adesso è il momento di mettere mano al file "playbook". Rispetto al file inventario che definisce le macchine nel concetto, passatemi il termine hardware, nel file playbook invece vengono definiti gli aspetti software, quindi si andranno a dichiarare tutti gli elementi software correlati.
Nel nostro esempio abbiamo decido di installare Nginx, il nostro primo file playbook quindi sarà: /etc/ansible/nginx-playbook.yml

Eccone il contenuto

    ---
    - hosts: webservers

    tasks:
      - ping: ~

      - name: Update APT package manager repositories cache
        become: true
        apt:
          update_cache: yes

      - name: Upgrade installed packages
        become: true
        apt:
          upgrade: safe

      - name: Install Nginx web server
        become: true
        apt:
         name: nginx
         state: latest

riuscite a intuire che operazioni eseguirà? dai è semplice e mi raccomando prestate attenzione all'identazione del file, è YAML lo sai....

Siamo pronti per provare il nostro primo playbook, per eseguirlo utilizzeremo questo comando

    sudo ansible-playbook nginx-playbook.yml

Ecco l'output che riceveremo

    PLAY [webservers] ******************************************************************************************************************************************************

    TASK [Gathering Facts] *************************************************************************************************************************************************
    ok: [192.168.1.154]
    ok: [192.168.1.150]
    ok: [192.168.1.149]

    TASK [ping] ************************************************************************************************************************************************************
    ok: [192.168.1.154]
    ok: [192.168.1.150]
    ok: [192.168.1.149]

    TASK [Update APT package manager repositories cache] *******************************************************************************************************************
    changed: [192.168.1.150]
    changed: [192.168.1.154]
    changed: [192.168.1.149]

    TASK [Upgrade installed packages] **************************************************************************************************************************************
    ok: [192.168.1.150]
    ok: [192.168.1.154]
    ok: [192.168.1.149]

    TASK [Install Nginx web server] ****************************************************************************************************************************************
    changed: [192.168.1.149]
    changed: [192.168.1.154]
    changed: [192.168.1.150]

    PLAY RECAP *************************************************************************************************************************************************************
    192.168.1.149              : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    192.168.1.150              : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    192.168.1.154              : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

    
**Grande Giove!**


    
