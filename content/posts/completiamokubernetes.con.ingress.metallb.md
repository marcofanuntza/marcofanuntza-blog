
---
title: "Completare cluster Kubernetes con Ingress NGINX, MetalLB e Cert-Manager"
date: 2025-05-24
description: "Guida passo passo per completare un cluster Kubernetes con Ingress Controller NGINX, MetalLB per il bilanciamento del carico e Cert-Manager per HTTPS automatico."
keywords: ["kubernetes", "ingress", "nginx", "metal-lb", "cert-manager", "helm", "kubespray", "ssl", "https", "devops"]
slug: completare-cluster-kubernetes
draft: false
cover:
  image: /img/install-ing-metal.webp
---

Nei miei post precedenti ho mostrato diversi metodi per installare un cluster Kubernetes: utilizzando Kind, K3D, Rancher e infine Kubespray.

Partendo proprio da quest’ultimo — l'articolo lo trovate qui: [Installiamo Kubernetes con Kubespray](https://marcofanuntza.it/posts/installiamo-kubernets-con-kubespray) — oggi voglio completare la configurazione del cluster aggiungendo tre componenti fondamentali per renderlo pronto a gestire ambienti di test e l’erogazione di servizi:

- **Ingress Controller** con NGINX  
- **MetalLB** per l’assegnazione di indirizzi IP ai LoadBalancer  
- **Cert-Manager** per la gestione automatica dei certificati SSL

Tutti questi componenti saranno installati tramite **Helm**, lo strumento di package management per Kubernetes.

> ⚠️ Per chi ha seguito l'articolo precedente, consideriamo come punto di partenza un cluster Kubernetes già funzionante.  
> Se invece il vostro cluster è stato creato con altri metodi, il risultato finale potrebbe differire, soprattutto a livello di configurazioni di rete e ruoli del cluster.

---

## A cosa servono questi componenti?

### Ingress Controller

Un *Ingress Controller* consente di esporre applicazioni interne al cluster verso l’esterno, utilizzando risorse di tipo `Ingress`.  
È fondamentale per gestire:

- l'accesso HTTP/HTTPS alle applicazioni,
- il routing per host o path,
- la gestione centralizzata dei certificati TLS.

In questo articolo utilizziamo il controller **NGINX**, uno dei più diffusi e mantenuti.

### MetalLB

In ambienti **bare-metal** o virtualizzati senza cloud provider, Kubernetes non può assegnare IP pubblici ai servizi `LoadBalancer`.  
**MetalLB** risolve questo problema offrendo funzionalità di bilanciamento del carico L2 o BGP e gestendo l’assegnazione di IP da un pool locale.

### Cert-Manager

**Cert-Manager** automatizza:

- la generazione,
- il rinnovo,
- e la distribuzione

di certificati TLS/SSL per le applicazioni esposte tramite Ingress. Supporta diverse *certificate authority* tra cui Let's Encrypt, e consente di avere HTTPS funzionante con il minimo sforzo.

---

## Installazione con Helm

### Installiamo Ingress Controller NGINX

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.kind=DaemonSet \
  --set controller.service.type=LoadBalancer
````

> Utilizziamo `DaemonSet` per deployare un controller su ogni nodo e `LoadBalancer` per permettere l’esposizione tramite MetalLB.

---

### Installiamo MetalLB

```bash
helm repo add metallb https://metallb.github.io/metallb
helm repo update

helm install metallb metallb/metallb \
  --namespace metallb-system \
  --create-namespace
```

Creiamo ora la configurazione `metal-lb-config.yaml` con il pool di IP da usare:

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default-address-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.240-192.168.1.250
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2-advertisement
  namespace: metallb-system
```

Applichiamo il file:

```bash
kubectl apply -f metal-lb-config.yaml
```

> 🔧 Personalizzate il range IP in base alla vostra rete locale.

---

### Installiamo Cert-Manager

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true
```

Ecco un esempio di `ClusterIssuer` per ottenere certificati da Let's Encrypt in modalità **staging**:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: tuo@email.it
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - http01:
        ingress:
          class: nginx
```

---

## Considerazioni finali

Con questi tre componenti — **Ingress NGINX**, **MetalLB** e **Cert-Manager** — il tuo cluster Kubernetes è finalmente pronto per:

* pubblicare applicazioni tramite Ingress,
* gestire indirizzi IP in ambienti non cloud,
* ottenere certificati HTTPS in modo automatico.

Questo è un setup ideale per ambienti **di test realistici**, **demo** o **piccole installazioni on-premise**.

Nel prossimo articolo potremmo vedere come deployare una vera applicazione web e renderla accessibile via HTTPS.

Hai dubbi, suggerimenti o vuoi raccontarmi la tua esperienza?
Lascia un commento o scrivimi!
