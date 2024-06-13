## Setting Up AWS Credentials as a Kubernetes Secret

This guide will walk you through the steps to create a Kubernetes secret for AWS credentials and configure a ProviderConfig to use these credentials.


### Steps
#### 1. Generate an AWS Key-Pair File

First, create a text file containing your AWS aws_access_key_id and aws_secret_access_key. The file should look like this:


```bash
[default]
aws_access_key_id = YOUR_ACCESS_KEY_ID
aws_secret_access_key = YOUR_SECRET_ACCESS_KEY
```
Save this file as aws-credentials.txt.

#### 2. Create a Kubernetes Secret with the AWS Credentials

To store the AWS credentials as a Kubernetes secret, use the following command:

```bash
kubectl create secret generic aws-secret -n crossplane-system --from-file=creds=./aws-credentials.txt
```

This command creates a secret named aws-secret in the crossplane-system namespace using the contents of the aws-credentials.txt file.
 Verify the Kubernetes Secret

You can verify that the secret has been created correctly by running:

```bash
kubectl describe secret aws-secret -n crossplane-system
```

#### 3. Create a ProviderConfig

Next, create a ProviderConfig to customize the settings of the AWS Provider. Apply the following Kubernetes configuration:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: aws.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: crossplane-provider-aws
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: aws-secret
      key: creds
EOF
```

This configuration links the AWS credentials, stored as a Kubernetes secret, to the ProviderConfig. The spec.credentials.secretRef.name value specifies the name of the Kubernetes secret (aws-secret), and the spec.credentials.secretRef.namespace specifies the namespace (crossplane-system).

By following these steps, you've successfully created a Kubernetes secret with your AWS credentials and configured the AWS Provider to use these credentials.