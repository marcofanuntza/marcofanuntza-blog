---
title: "Sincronizzazione Indici Elasticsearch con Elasticdump in Kubernetes"
date: 2025-05-22
description: "Come sincronizzare gli indici Elasticsearch tra due istanze in Kubernetes usando Elasticdump."
tags: ["kubernetes", "elasticsearch", "elasticdump", "devops"]
draft: false
cover:
  image: /img/helmchart-elastic.webp
---

## Eseguire un'immagine temporanea nello stesso namespace

Il comando seguente consente di creare un **pod temporaneo** nel namespace `elasticsearch-prod`, ed accedere alla sua shell Bash:

```bash
kubectl run elasticdump-client --rm -it --image=node:18 -n elasticsearch-prod -- bash
````

## Installare Elasticdump

Una volta nella shell del pod, installiamo `elasticdump` tramite `npm`:

```bash
npm install -g elasticdump
```

---

## Eseguire la sincronizzazione degli indici

Ora siamo pronti per eseguire il **sync** dei dati tra gli indici. Esegui il seguente comando per ogni indice da copiare:

### Indice: `prod-index`

```bash
NODE_TLS_REJECT_UNAUTHORIZED=0 elasticdump \
  --input=http://elasticsearch-old-prod.elasticsearch-old-prod.svc.cluster.local:9200/prod-index \
  --output=https://elastic:password@elasticsearch-new-prod.elasticsearch-new-prod.svc.cluster.local:9200/prod-index \
  --type=data
```


---

## Analisi del comando

* `NODE_TLS_REJECT_UNAUTHORIZED=0`: disabilita il controllo del certificato SSL. Necessario perché l'istanza di destinazione ha **X-Pack Security** abilitato e accetta solo connessioni HTTPS.

* `--input=...`: URL della sorgente Elasticsearch che contiene l’indice da copiare.

* `--output=...`: URL della destinazione Elasticsearch. Verrà creato (o aggiornato) un indice con lo stesso nome e contenuti.

---

> ⚠️ **Nota di sicurezza**: Disabilitare la verifica SSL è sconsigliato in ambienti di produzione pubblici. Utilizzare solo in ambienti interni o di sviluppo.

```
