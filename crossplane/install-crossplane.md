## Install crossplane


```
helm repo add crossplane-stable https://charts.crossplane.io/stable



helm install crossplane --namespace crossplane-system --create-namespace crossplane-stable/crossplane -f values.yaml
```


install provider in provider directory


create a secret in crossplane namespace

kubectl create secret generic aws-secret -n crossplane-system --from-file=creds=./aws-credentials.txt

aws-credentials.txt

[default]

aws_access_key_id = 

aws_secret_access_key =