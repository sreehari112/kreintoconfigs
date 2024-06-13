# Install vault ha with raft storage using helm chart

Please make helm chart is installed

```
helm repo add hashicorp https://helm.releases.hashicorp.com

```

```
helm repo update
```

```
helm search repo hashicorp/vault
```

vaules.yaml

```

```
server:
  affinity: ""
  ha:
    enabled: true
    raft: 
      enabled: true
ui:
 enabled: true
 serviceType: "ClusterIP"
```

```
helm install vault hasicorp/vault --version 1.16.1 --create-namespace -n vault -f values.yaml
```

This creates three Vault server instances with an Integrated Storage (Raft) backend

Display all the pods within the vault namespace.

```
kubectl get po -n vault
```

Initialize vault-0 to generate vault unseal keys and root token.

```
 kubectl exec vault-0 -n vault -- vault operator init -format=json > cluster-keys.json
```

```
cat cluster-keys.json
```

select each base64 encode key from unseal_keys_b64 from above file. Default shared keys will be 5. You need to execute the steps 5 times in each vault pod. Start with vault-0


kubectl -n vault exec vault-0 -- vault operator unseal <UNSEAL KEY 1>
kubectl -n vault exec vault-0 -- vault operator unseal <UNSEAL KEY 2>
kubectl -n vault exec vault-0 -- vault operator unseal <UNSEAL KEY 3>
kubectl -n vault exec vault-0 -- vault operator unseal <UNSEAL KEY 4>
kubectl -n vault exec vault-0 -- vault operator unseal <UNSEAL KEY 5>

Once above commands are completed executing in vault-0 pod. Join the vault-1 and vault-2pods to the Raft cluster

```
kubectl exec -ti vault-1 -n vault -- vault operator raft join http://vault-0.vault-internal:8200

kubectl exec -ti vault-2 -n vault -- vault operator raft join http://vault-0.vault-internal:8200

```

Use the unseal key from above to unseal vault-1 and vault-2


After this unsealing process all vault pods are now in running (1/1 ready ) state

```
kubectl get po -n vault
```


Vault service is of ClusterIP type which means we can access Vault console from browser so to access this we need to use port-forward command

```
kubectl port-forward service/vault -n vault 8200:8200

```

Create ingressroute using below

ingress.yaml
```

```