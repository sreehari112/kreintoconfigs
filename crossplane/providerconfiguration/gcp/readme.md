## Setting Up GCP Credentials as a Kubernetes Secret

This guide will walk you through the steps to create a Kubernetes secret for GCP credentials and configure a ProviderConfig to use these credentials.
Steps

### 1. Generate a GCP Service Account JSON File

First, create a GCP Service Account and download the JSON key file. Follow these steps:

1. Go to the [GCP Console](https://console.cloud.google.com/).

2. Navigate to the IAM & Admin > Service Accounts section.

3. Click on "Create Service Account".
    
4. Fill in the required details and click "Create".

5. Assign the necessary roles to the service account and click "Continue".

6. Click on "Create Key", select "JSON", and download the key file.

Save this JSON file as gcp-credentials.json.

or

```bash
export PROJECT_ID=crossplane-426011
export SA_NAME=crossplane-master-sa
export SA="${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"
gcloud iam service-accounts create $SA_NAME --project $PROJECT_ID
gcloud projects add-iam-policy-binding --role roles/admin $PROJECT_ID --member serviceAccount:$SA
gcloud iam service-accounts keys create gcp-credentials.json --project $PROJECT_ID --iam-account $SA
```

### 2. Create a Kubernetes Secret with the GCP Credentials

A Kubernetes generic secret has a name and contents. Use the following command to generate a secret object named gcp-secret in the crossplane-system namespace:

```bash
kubectl create secret generic gcp-secret -n crossplane-system --from-file=creds=./gcp-credentials.json
```

This command creates a secret named gcp-secret in the crossplane-system namespace using the contents of the gcp-credentials.json file.

### 3. Verify the Kubernetes Secret

You can verify that the secret has been created correctly by running:

```bash
kubectl describe secret gcp-secret -n crossplane-system
```
The output should look similar to this:

```plaintext

Name:         gcp-secret
Namespace:    crossplane-system
Labels:       <none>
Annotations:  <none>
Type:         Opaque
Data
====
creds:  2330 bytes
```


### 4. Create a ProviderConfig

A ProviderConfig customizes the settings of the GCP Provider. Include your GCP project ID in the ProviderConfig settings.

:bulb:**Tip: Find your GCP project ID from the project_id field of the gcp-credentials.json file.**


Apply the ProviderConfig with the following command:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: gcp.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: crossplane-provider-gcp
spec:
  projectID: YOUR_GCP_PROJECT_ID
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: gcp-secret
      key: creds
EOF
```

Replace YOUR_GCP_PROJECT_ID with your actual GCP project ID.

This configuration links the GCP credentials, stored as a Kubernetes secret, to the ProviderConfig. The spec.credentials.secretRef.name value specifies the name of the Kubernetes secret (gcp-secret), and the spec.credentials.secretRef.namespace specifies the namespace (crossplane-system).

By following these steps, you've successfully created a Kubernetes secret with your GCP credentials and configured the GCP Provider to use these credentials.