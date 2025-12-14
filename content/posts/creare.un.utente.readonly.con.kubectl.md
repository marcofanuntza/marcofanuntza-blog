---
title: "Creare un utente Kubernetes read-only con kubectl"
date: 2025-12-14
draft: false
keywords: ["Kubernetes", "RBAC", "DevOps"]
tags: ["kubectl", "read-only", "certificati", "RBAC"]
cover:
  image: /img/install-ing-metal.webp
---


# Creare un utente **read-only** in Kubernetes (solo kubectl, senza OIDC)

In questo articolo vediamo come creare **uno user Kubernetes con accesso esclusivamente in lettura** utilizzando **certificati client x509** e **RBAC**, senza Keycloak, senza dashboard, solo Kubernetes puro.

La procedura è valida per cluster installati con **kubespray** o kubeadm ed è adatta a:

* utenti di consultazione
* operatori di supporto
* audit / troubleshooting

---

## Obiettivo

Lo user finale potrà:

* `get`, `list`, `watch`
* usare `kubectl get`, `describe`, `logs`

Non potrà:

* creare / modificare / cancellare risorse
* usare `kubectl apply`, `delete`, `exec`

---

## Concetti chiave

Kubernetes **non gestisce utenti interni**.
L’accesso avviene in due fasi:

1. **Autenticazione** → certificato client
2. **Autorizzazione** → RBAC (Role / ClusterRole)

---

## 1. Creazione della chiave privata

```bash
openssl genrsa -out user-readonly.key 2048
```

---

## 2. Creazione della CSR

Il `CN` diventerà lo username visto da Kubernetes.

```bash
openssl req -new \
  -key user-readonly.key \
  -out user-readonly.csr \
  -subj "/CN=user-readonly/O=readonly"
```

---

## 3. Firma del certificato con la CA del cluster

Da eseguire su un control-plane:

```bash
openssl x509 -req \
  -in user-readonly.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -CAcreateserial \
  -out user-readonly.crt \
  -days 365
```

---

## 4. Creazione della ClusterRole read-only

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

---

## 5. Binding dello user alla ClusterRole

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

---

## 6. Creazione del kubeconfig dedicato

Si utilizza un kubeconfig **file-based** per ridurre gli errori.

```yaml
apiVersion: v1
kind: Config

clusters:
- name: kubernetes
  cluster:
    server: https://IP-API-SERVER:6443
    certificate-authority: ./ca.crt

users:
- name: user-readonly
  user:
    client-certificate: ./user-readonly.crt
    client-key: ./user-readonly.key

contexts:
- name: readonly-context
  context:
    cluster: kubernetes
    user: user-readonly

current-context: readonly-context
```

---

## 7. Test dell’accesso

```bash
KUBECONFIG=./kubeconfig-readonly kubectl get pods -A
```

Test di un’operazione vietata:

```bash
KUBECONFIG=./kubeconfig-readonly kubectl delete pod test -n default
```

Risultato atteso:

```
Error from server (Forbidden)
```

---

## 8. Verifica RBAC

```bash
kube
```
