---
title: Grazie a Pi-Hole basta con la pubblicità!
author: Marco Fanuntza
date: 2024-01-06T09:53:54+00:00
draft: false
url: /2024/01/06/grazie-pihole-basta-pubblicita/

categories:
  - linux
  - server
tags:
  - adblocker
  - advertising
  - Cluster
  - Gestione Server
  - Linux Containers
  - pi-hole
  - pihole
  - Virtualizzazione

cover:
    image: /img/pi-hole-logo.webp
---

Quanto vi infastidiscono gli annunci pubblicitari navigando sul web? Ormai ci sono pagine web piene di annunci, pop-up fastidiosissimi che non fanno altro che farci perdere tempo e voglia di visitarne il sito, i quotidiani con le news e le notizie sportive su tutti sono i più scassa bit.

Per fortuna ci viene in aiuto Pi-Hole!

![Example image](/img/pi-hole-pagina0.webp)


Pi-hole è un software open-source progettato per il controllo e la gestione della rete orientato nello specifico proprio per combattere la pubblicità e gli annunci correlati, sostanzialmente agisce come un filtro DNS, offrendo funzionalità avanzate per bloccare tutti gli annunci pubblicitari, tracker e tutti i contenuti indesiderati ancor prima che raggiungano i nostri dispositivi connessi alla rete.

**Caratteristiche Principali:**

**Blocco degli Annunci**: Pi-hole utilizza liste di blocco per intercettare le richieste DNS associate agli annunci pubblicitari, consentendo di eliminare la visualizzazione di annunci indesiderati su tutti i dispositivi connessi alla rete.

**Filtro DNS**: Attraverso il blocco delle richieste DNS indesiderate, Pi-hole inoltre impedisce l&#8217;accesso a siti malevoli contenenti malware o altri contenuti indesiderati, contribuendo a migliorare la sicurezza della navigazione.

**Monitoraggio delle Prestazioni**: Pi-hole fornisce statistiche dettagliate sulle richieste DNS e sui domini bloccati, permettendo agli utenti di monitorare l&#8217;attività di rete e l&#8217;efficacia dei blocchi.

**Interfaccia Web**: Il software è dotato di un&#8217;interfaccia utente web che semplifica la configurazione e la gestione. Attraverso questa interfaccia, gli utenti possono monitorare le statistiche, aggiornare liste di blocco e personalizzare le impostazioni.

**Liste di Blocco Personalizzabili**: Pi-hole consente agli utenti di aggiungere o rimuovere domini dalle liste di blocco in base alle proprie preferenze e esigenze.

**Supporto per IPv6**: Il software supporta IPv6, garantendo una copertura completa delle richieste DNS e dei blocchi su entrambi gli standard di indirizzamento IP.

**Integrazione con DHCP**: Pi-hole può fungere anche da server DHCP, semplificando ulteriormente la gestione degli indirizzi IP e la distribuzione delle configurazioni di rete.



Viste le caratteristiche non poteva mancare nella suite di servizi installati sul mio homelab, l&#8217;installazione di per sè é molto semplice infatti basterà eseguire un semplice script che vi auto guiderà nelle varie impostazioni

    curl -sSL https://install.pi-hole.net | bash


Io personalmente per ora l&#8217;ho installato su un raspberry Pi 3 dove sono già presenti le istanze di Nginx Proxy Manager e il container per la gestione del tunnel cloudflare, scriverò un&#8217;articolo specifico su questa macchina in futuro.

Voi potete installarlo dove meglio credete, un server linux, un container docker o su una VM, up to you! La documentazione è molto chiara e completa per ogni dettaglio specifico non posso che invitarvi a leggerla cliccando [QUI](https://docs.pi-hole.net/)

Attenzione è importante che il server abbia l&#8217;indirizzo IP statico, questo non dovrà cambiare mai perché verrà chiamato da tutti gli altri device, se avete il DHCP attivo basterà impostare una reservation.

Ecco l'aspetto che ha la comodissima interfaccia WEB
![Example image](/img/pi-hole-pagina1.webp)

Oltre a liberami dalla pubblicità Pi-Hole mi è stato utilissimo anche come DNS locale per il mio homelab

![Example image](/img/pi-hole-pagina2.webp) 

Per completare il tutto non dovete fare altro che modificare i parametri network sui vostri device facendo in modo che il DNS utilizzato sia l&#8217;IP della vostra installazione Pi-Hole, in questo modo ogni richiesta verrà gestita da Pi-Hole e addio alla pubblicità!

Per chi ne avesse la possibilità la modifica DNS si potrebbe eseguire già a livello del vostro router, basterà editare una sola volta sull&#8217;apparato per avere così di conseguenza tutti i device già configurati.

Evviva l&#8217;opensource!