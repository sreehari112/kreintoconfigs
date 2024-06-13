# Creating and Managing Secrets in HashiCorp Vault for Kubernetes Deployments

### Table of Contents:
    1. Introduction
    2. Creating and Managing Secrets
      * Key/Value Secrets Engine
      * Storing Secrets
      * Reading Secrets
      * Updating Secrets
      * Deleting Secrets
    3. Integrating Vault with Kubernetes
      * Enabling Kubernetes Auth Method
      * Configuring Vault Policies
      * Creating Kubernetes Service Account for Vault
      * Configuring Kubernetes to Use Vault Secrets
    4. Vault Administration
      * Policy Management
      * Auditing
      * Backup and Restore


## 1. Introduction

HashiCorp Vault is a powerful tool for managing secrets, providing a unified interface to any secret while enforcing access control and recording a detailed audit log. This guide focuses on creating and managing secrets using Vault's Key/Value secrets engine, integrating these secrets with Kubernetes, and performing necessary Vault administration tasks.


## 2. Creating and Managing Secrets

To manage secrets that can be consumed by Kubernetes deployments, we need to enable and use the KV secrets engine.

### Key/Value Secrets Engine

The Key/Value secrets engine allows you to store arbitrary secrets as key-value pairs. We'll enable it at a specified path.


### Enable the KV Secrets Engine

```bash
kubectl -n sys-vault exec -it vault-0 -- vault secrets enable -path=secret kv
```
This command enables the KV secrets engine at the path secret/. Here, vault-0 is assumed to be the leader Vault pod.

### Storing Secrets

To store secrets, create a path following the convention: secret/APPLICATIONNAME/FILENAME.

*Example Path: secret/myapp/config*

From the above path secret will be the root path which should not be modified. APPLICATIONNAME and FILENAME will the application name who want to store the secrets and filename will be the configuration file where applications configuration is saved.


```bash
kubectl -n sys-vault exec -it vault-0 -- vault kv put secret/myapp/config username='myuser' password='mypassword'
```
This command stores the username and password for the myapp application

### Reading Secrets

Read secrets from the KV secrets engine:


```bash
kubectl -n sys-vault exec -it vault-0 -- vault kv get secret/myapp/config
```
This command retrieves the secrets stored at the path secret/myapp/config.

### Updating Secrets

Update existing secrets in the KV secrets engine:


```bash
kubectl -n sys-vault exec -it vault-0 -- vault kv put secret/myapp/config username='myuser' password='newpassword'
```
This command updates the password for the myapp application.

### Deleting Secrets

Delete secrets from the KV secrets engine:


```bash
kubectl -n sys-vault exec -it vault-0 -- vault kv delete secret/myapp/config
```
This command deletes the secrets stored at the path secret/myapp/config.


## 3. Integrating Vault with Kubernetes

### Enabling Kubernetes Auth Method


To allow Kubernetes to authenticate with Vault, we need to enable the Kubernetes authentication method in Vault.


```bash
kubectl -n sys-vault exec -it vault-0 -- vault auth enable kubernetes
```
Configure the Kubernetes authentication method with your cluster details:


```bash
kubectl -n sys-vault exec -it vault-0 -- vault write auth/kubernetes/config \
    token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
    kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
    kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

### Configuring Vault Policies:

Policies define what actions are allowed. We'll create a policy to allow read access to our application's secrets.

Create Policy File: 


```bash
# create following file myapp-policy.hcl
path "secret/data/myapp/config" {
  capabilities = ["read"]
}
```
Apply the policy in Vault:
```bash
kubectl -n sys-vault exec -it vault-0 -- vault policy write myapp-policy myapp-policy.hcl
```

#### Creating Kubernetes Service Account for Vault

Create a Kubernetes service account and role binding to allow Vault to authenticate.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: vault-auth
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: vault-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: vault-auth-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: vault-auth
  namespace: default
roleRef:
  kind: Role
  name: vault-reader
  apiGroup: rbac.authorization.k8s.io
```

### Configuring Kubernetes to Use Vault Secrets

Use the Vault Agent Injector to automatically inject secrets into your Kubernetes pods. Add annotations to your pod template:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "myapp-role"
        vault.hashicorp.com/agent-inject-secret-config: "secret/data/myapp/config"
    spec:
      serviceAccountName: vault-auth
      containers:
      - name: myapp
        image: myapp:latest
        env:
          - name: USERNAME
            valueFrom:
              secretKeyRef:
                name: vault-secrets
                key: username
          - name: PASSWORD
            valueFrom:
              secretKeyRef:
                name: vault-secrets
                key: password
```
## 4. Vault Administration

### Policy Management

Policies define what actions users and applications can perform. Use HCL or JSON to write policies and apply them:

```bash
# myadmin-policy.hcl
path "sys/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}
```

Apply the policy:
```bash
kubectl -n sys-vault exec -it vault-0 -- vault policy write myadmin-policy myadmin-policy.hcl
```
### Auditing

Enable audit logging to keep track of all requests to Vault:

```bash
kubectl -n sys-vault exec -it vault-0 -- vault audit enable file file_path=/var/log/vault_audit.log
```
### Backup and Restore: 

Backup Vault data regularly to avoid data loss:
```bash
kubectl -n sys-vault exec -it vault-0 -- vault operator raft snapshot save backup.snap
```

Restore Vault data from a snapshot:
```bash
kubectl -n sys-vault exec -it vault-0 -- vault operator raft snapshot restore backup.snap
```

### Automatic backups
