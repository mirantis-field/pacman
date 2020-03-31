# Pac-Man Node.js App On Single Kubernetes Cluster

This guide will walk you through creating a single Kubernetes cluster and deploy the Pac-Man Node.js application onto it.

## Setup Environment Variables

```bash
export DKS_NAMESPACE=pacman
```

### Source Your Client Bundle

```bash
cd client-bundle
source env.sh
```

## Create MongoDB Resources

### Ensure a Default StorageClass is Configured

```bash
kubectl get storageclass --namespace=${DKS_NAMESPACE}
```

### Create MongoDB Persistent Volume Claim

We need to create persistent volume claim for our MongoDB to persist the database.

```bash
kubectl create --filename=persistentvolumeclaim/mongo-pvc.yaml --namespace=${DKS_NAMESPACE}
```

Wait until the pvc is bound:

```bash
kubectl get persistentvolumeclaims mongo-storage --namespace=${DKS_NAMESPACE} --output=wide --watch
```

### Create MongoDB Deployment

Now create the MongoDB deployment that will use the `mongo-storage` persistent volume claim to mount the directory that is to contain the MongoDB database files.

```bash
kubectl create --filename=deployments/mongo-deployment.yaml --namespace=${DKS_NAMESPACE}
```

Scale the deployment, since the deployment definition has replicas set to 0:

```bash
kubectl scale deployment mongo --namespace=${DKS_NAMESPACE} --replicas=1
```

Verify the container has been created and is in the running state:

```bash
kubectl get pods --namespace=${DKS_NAMESPACE} --output=wide --watch
```

### Create MongoDB Service

This component creates a mongo DNS entry, so this is why we use `mongo` as the host we connect to in our application instead of `localhost`.

```bash
kubectl create --filename=services/mongo-service.yaml --namespace=${DKS_NAMESPACE}
```

Wait until the mongo service has the external IP address listed:

```bash
kubectl get service mongo --namespace=${DKS_NAMESPACE} --output=wide --watch
```

## Creating the Pac-Man Resources

### Create Pac-Man Deployment

Now create the Pac-Man deployment.

```bash
kubectl create --filename=deployments/pacman-deployment.yaml --namespace=${DKS_NAMESPACE}
```

Let's describe the Pac-Man deployment.

```bash
kubectl describe deployment pacman --namespace=${DKS_NAMESPACE}
```

Scale the deployment, since the deployment definition has replicas set to 0.

```bash
kubectl scale deployment pacman --namespace=${DKS_NAMESPACE} --replicas=2
```

Verify the containers have been created and are in the running state:

```bash
kubectl describe deployment pacman --namespace=${DKS_NAMESPACE}
```

```bash
kubectl get pods --namespace=${DKS_NAMESPACE} --output=wide --watch
```

### Create Pac-Man Service

This component creates the service to access the application.

```bash
kubectl create --filename=services/pacman-service.yaml --namespace=${DKS_NAMESPACE}
```

Wait until the pacman service has the external IP address listed:

```bash
kubectl get service pacman --namespace=${DKS_NAMESPACE} --output=wide --watch
```

Once the pacman pods are running and the `pacman` service has an IP address, open up your browser and try to access it via `http://<EXTERNAL_IP>/`.

## Cleanup

### Delete Kubernetes cluster

Delete the GKE cluster.

```
gcloud container --project "<YOUR_PROJECT_ID>" \
  clusters delete "kube-ref-app" --zone "us-central1-f"
```
