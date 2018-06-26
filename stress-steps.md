## Deploy Cluster

```bash
gcloud beta container clusters create meetup-prebake \
--node-locations us-central1-a,us-central1-b,us-central1-c \
--zone us-central1-a \
--cluster-version 1.10.4-gke.0 \
--machine-type g1-small \
--num-nodes 2 \
--enable-autoscaling \
--min-nodes 2 \
--max-nodes 6 \
--enable-cloud-monitoring \
--enable-autorepair \
--enable-stackdriver-kubernetes
```

## Add a PVM pool

```bash
gcloud container node-pools create pvm-pool \
  --cluster meetup-prebake \
  --preemptible \
  --zone us-central1-a \
  --enable-autoupgrade \
  --num-nodes 1 \
  --machine-type g1-small \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=6
```

## Connect to Cluster

```bash
gcloud container clusters get-credentials meetup-prebake --zone us-central1-a
```

## Deploy Stress

```bash
kubectl create -f stress-deployment.yaml
```

## Scale the deployment
```bash
kubectl scale deployment stress --replicas=10
```

## Deploy Stress to PVM only nodes

```bash
kubectl create -f stress-pvm-deployment.yaml
```