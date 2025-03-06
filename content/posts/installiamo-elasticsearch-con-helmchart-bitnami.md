---
title: "Installare Elasticsearch con Helm Chart di Bitnami"
date: 2025-02-26
description: "Guida rapida per installare Elasticsearch su Kubernetes utilizzando HelmChart di Bitnami."
categories: ["Kubernetes", "Helm", "Elasticsearch"]
tags: ["Helm", "Bitnami", "Elasticsearch", "Kubernetes"]
draft: false
cover:
  image: /img/helmchart-elastic.webp
---

## Introduzione
In questa guida vedremo come installare **Elasticsearch** su Kubernetes utilizzando **HelmChart di Bitnami**. Seguiremo un approccio strutturato, includendo la configurazione personalizzata tramite un file `values.yaml`

## Aggiungere il repository Helm di Bitnami
Per prima cosa, aggiungiamo il repository **Bitnami** alla nostra installazione di Helm:

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Se il repository è già stato aggiunto in precedenza, aggiorniamolo per assicurarci di avere la versione più recente:

```sh
helm repo update
```

## Verificare le versioni disponibili di Elasticsearch
Se abbiamo bisogno di installare una versione specifica di **Elasticsearch**, possiamo controllare le versioni disponibili nel repository:

```sh
helm search repo bitnami/elasticsearch --versions
```

## Creare un namespace dedicato
Per una migliore organizzazione, creiamo un namespace dedicato per la nostra installazione di Elasticsearch:

```sh
kubectl create ns elasticsearch-test-stage
```

## Configurare Elasticsearch con un file `values.yaml`
Possiamo personalizzare l'installazione specificando alcuni parametri in un file YAML. Ecco un esempio di configurazione che utilizzeremo:

```yaml
coordinating:
  replicaCount: 1
data:
  replicaCount: 1
global:
  kibanaEnabled: true
ingest:
  enabled: false
ingress:
  enabled: true
  ingressClassName: nginx
master:
  replicaCount: 1
```

⚠️ **Attenzione:** Assicurati che il file sia correttamente indentato per evitare errori YAML.

## Installare Elasticsearch con Helm
Ora possiamo eseguire l'installazione specificando la versione desiderata e il file di configurazione:

```sh
helm install elasticsearch-test-stage bitnami/elasticsearch \
  --version 19.9.4 \
  --namespace elasticsearch-test-stage \
  -f values.yaml
```

L'installazione potrebbe richiedere alcuni minuti. 

## Verificare l'installazione
Una volta completata l'installazione, possiamo verificare che tutto sia stato creato correttamente:

```sh
kubectl get all -n elasticsearch-test-stage
```

Se tutto è andato a buon fine, Elasticsearch sarà pronto per l'uso! 🚀


tips. terminata installazione vi siete resi conto di aver dimenticato un ulteriore parametro necessario? La soluzione è semplicissima!

Editate il file values.yaml e invece che utilizzare "install" eseguirete il comando con "upgrade" 

```sh
helm upgrade elasticsearch-test-stage bitnami/elasticsearch \
  --version 19.9.4 \
  --namespace elasticsearch-test-stage \
  -f values.yaml
```

Happy helming!

---


