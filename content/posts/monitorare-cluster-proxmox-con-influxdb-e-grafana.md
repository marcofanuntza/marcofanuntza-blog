---
title: "Come monitoro il mio cluster Proxmox con InfluxDB e Grafana"
date: 2024-12-27T08:59:48+01:00
draft: false

categories:
 - Howto
 - Proxmox
 - Grafana

tags:
 - Grafana
 - Proxmox
 - Monitoring
 - Portainer
 - Docker
 - Influxdb
 - Tutorial

cover:
  image: /img/monit-inluxdb7.webp
---

Nel mio Homelab, come ho descritto in precedenza in altri post, utilizzo un cluster Proxmox. 

Sebbene il cluster disponga già di una dashboard integrata, questa risulta piuttosto spartana. 
Così, in questo tranquillo pomeriggio delle ferie natalizie, ho colto l'occasione per sperimentare una soluzione più avanzata: InfluxDB + Grafana.


**InfluxDB** è un database open-source progettato specificamente per la gestione di serie temporali, ovvero dati che variano nel tempo come metriche, eventi e log. Grazie alla sua architettura ottimizzata, è in grado di gestire grandi volumi di dati in tempo reale, rendendolo una scelta ideale per applicazioni di monitoraggio e analisi delle prestazioni.  

Grazie alla sua natura scalabile e alla capacità di integrarsi con strumenti come Grafana, Prometheus e Telegraf, InfluxDB è ampiamente utilizzato nel campo DevOps. Con un'interfaccia semplice e un set di funzionalità avanzate, rappresenta una soluzione potente per chiunque abbia bisogno di analizzare e visualizzare dati basati sul tempo.


**Grafana** è una piattaforma open-source progettata per la visualizzazione e l'analisi di dati in tempo reale, utilizzata principalmente per il monitoraggio delle metriche e dei log. Grazie alla sua capacità di supportare numerose fonti di dati, come Prometheus, InfluxDB, Elasticsearch e database relazionali, consente agli utenti di creare dashboard altamente personalizzabili e interattive per analizzare informazioni complesse.

Oltre alla visualizzazione, Grafana offre funzionalità avanzate di alerting, permettendo di configurare avvisi personalizzati in base a specifiche condizioni e inviarli tramite email, Slack o altri canali. 


**Il paradigma è semplice** >> **Proxmox** genera le metriche, **InfluxDB** si occupa di salvare le metriche in un database (bucket) e **Grafana** infine restituirà le stesse sottoforma di dashboard con grafici dettagliati.


**Per installare** InfluxDB e Grafana ho deciso di procedere utilizzando dei container Docker e per comodità ho scelto un'approccio molto easy & lazy... tradotto significa.. utilizzo Portainer e suoi templates. Basta semplicemente andare su Templates e nella ricerca scrivere influxdb e poi grafana, troverete rispettivamente: **InfluxDB for Edge** e **Grafana Dashboard**

I template su Portainer sono già strutturati per l'inserimento delle variabili, nello specifico per InfluxDB ci sarà la creazione di un primo bucket (Influx Bucket Name) e una organization (Influx Org Name), questi dati saranno utili nel passaggio successivo. 

Dopo aver installato entrambi vai istruito il cluster per indirizzare le metriche su InfluxDB, su Proxmox dalla GUI sulla sezione root del Datacenter si va sul menù **Metric Server** ADD > InfluxDB

![Example image](/img/monit-inluxdb1.webp)

Come potete notare viene chiesto anche un Token, questo va creato su InfluxDB, io per comodità ho clonato il token admin già presente su sezione Load Data >> API Tokens 

![Example image](/img/monit-inluxdb2.webp)

Salvate su Proxmox e se i dati sono stati inseriti correttamente dovreste già vedere su InfluxDB nella sezione Data Explorer il bucket popolarsi con le metriche, nel mio caso il bucket si chiama proxmox-mnt.

![Example image](/img/monit-inluxdb3.webp)
 
La parte di configurazione su Proxmox e InfluxDB possiamo considerarla conclusa, ora si potrà continuare su Grafana.

Su Grafana iniziamo con "agganciare" la base dati salvata da InfluxDB, per fare questo rechiamoci sulla sezione Connections e poi Data Sources, qui nella ricerca inseriamo InfluxDB come sorgente e compiliamo i dati necessari

![Example image](/img/monit-inluxdb4.webp)

Prestate attenzione nel linguaggio SQL da scegliere, e importante selezionare Flux

![Example image](/img/monit-inluxdb5.webp)

Completiamo inserendo i dati per far raggiungere InfluxDB e relativo bucket, il token che andremo a utilizzare si è lo stesso di prima. Salvate con test positivo.

![Example image](/img/monit-inluxdb6.webp)

Adesso arriva la parte finale e più soddisfacente, su Grafana dobbiamo creare la nostra Dashboard. Cioè creare... c'è già qualcuno più bravo di noi e disponibile che ha già fatto una dashboard! 

Grazie alla community Grafana permette di condividere dashboard già pronte e semplici da importare direttamente sulla vostra istanza, queste si trovano facilmente sul sito grafanalabs.

![Example image](/img/monit-inluxdb9.webp)

Provate a cercare proxmox

![Example image](/img/monit-inluxdb10.webp)




