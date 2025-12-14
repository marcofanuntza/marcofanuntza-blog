---
title: "Creare un utente Kubernetes read-only per kubectl"
date: 2025-12-14
draft: false
keywords: ["Kubernetes", "RBAC", "DevOps"]
tags: ["kubectl", "read-only", "certificati", "RBAC"]
cover:
  image: /img/read-only-user.webp
---

# Creare un utente **read-only** in Kubernetes

In questo post riassumo le operazioni per creare **uno user Kubernetes con accesso in sola lettura** utilizzando **certificati client x509** e **RBAC**.

La procedura è valida per qualsiasi cluster Kubernetes

## Obiettivo

Lo user read-only potrà eseguire operazioni di:

* `get`, `list`, `watch`
* usare `kubectl get`, `describe`, `logs`

Non potrà:

* creare / modificare / cancellare risorse
* usare `kubectl apply`, `delete`, `exec`

## Concetti chiave

Kubernetes nativamente **non gestisce utenti interni**.

L’accesso è possibile grazie a due componenti:

* **Autenticazione** → certificato client
* **Autorizzazione** → RBAC (Role / ClusterRole)

## Creazione della chiave privata

Per comodità tutte le operazioni che seguono verranno eseguite da shell su un nodo control-plane del vostro cluster Kubernetes

```bash
openssl genrsa -out user-readonly.key 2048
```

## Creazione della CSR

Il `CN` diventerà lo username visto da Kubernetes.

```bash
openssl req -new \
  -key user-readonly.key \
  -out user-readonly.csr \
  -subj "/CN=user-readonly/O=readonly"
```

## Firma del certificato con la CA del cluster


```bash
openssl x509 -req \
  -in user-readonly.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -CAcreateserial \
  -out user-readonly.crt \
  -days 365
```

## Creazione della ClusterRole read-only

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: readonly-cluster
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]
```

```bash
kubectl apply -f readonly-clusterrole.yaml
```

## Binding dello user alla ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: readonly-user-binding
subjects:
- kind: User
  name: user-readonly
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: readonly-cluster
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f readonly-binding.yaml
```

## Creazione del kubeconfig dedicato (metodo base64)

Qui utilizziamo il metodo con i certificati codificati in Base64 direttamente nel kubeconfig.

1. Converti i certificati in Base64:

```bash
base64 -w0 user-readonly.crt > user-readonly.crt.base64
base64 -w0 user-readonly.key > user-readonly.key.base64
base64 -w0 /etc/kubernetes/pki/ca.crt > ca.crt.base64
```

2. Crea il kubeconfig:

```yaml
apiVersion: v1
kind: Config
clusters:
- name: kubernetes
  cluster:
    server: https://IP-API-SERVER:6443
    certificate-authority-data: <contenuto di ca.crt.base64>

users:
- name: user-readonly
  user:
    client-certificate-data: <contenuto di user-readonly.crt.base64>
    client-key-data: <contenuto di user-readonly.key.base64>

contexts:
- name: readonly-context
  context:
    cluster: kubernetes
    user: user-readonly

current-context: readonly-context
```

Sostituisci `<contenuto di ...>` con i rispettivi file Base64 generati.

## Test dell’accesso

```bash
KUBECONFIG=./kubeconfig-readonly kubectl get nodes
```

Che ci mostrerà:

```bash
NAME                  STATUS   ROLES           AGE   VERSION
kubespray-master-01   Ready    control-plane   21d   v1.34.2
kubespray-master-02   Ready    control-plane   21d   v1.34.2
kubespray-master-03   Ready    control-plane   21d   v1.34.2
kubespray-worker-01   Ready    <none>          21d   v1.34.2
kubespray-worker-02   Ready    <none>          21d   v1.34.2
kubespray-worker-03   Ready    <none>          21d   v1.34.2
```

Test di un’operazione vietata:

```bash
KUBECONFIG=./kubeconfig-readonly kubectl delete pod test -n default
```

Che se tutto è andato per il verso giusto ci sarà errore:

```
Error from server (Forbidden): pods "test" is forbidden: User "user-readonly" cannot delete resource "pods" in API group "" in the namespace "default"
```

## Ulteriore verifica tramite RBAC

```bash
kubectl auth can-i get pods --as=user-readonly
kubectl auth can-i delete pods --as=user-readonly
```

## Conclusione

Questo approccio fornisce un accesso **minimale, sicuro e nativo** a Kubernetes, ideale per ambienti enterprise o cluster senza Identity Provider esterni.

*Happy kubectl!* 🚀
