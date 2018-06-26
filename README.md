# Kubernetes meet-up: cost-effective Kubernetes DEMO
_How-to decrease cost with Preemptible Compute Engine Machines_

<br />

----------------------------------------------------------------------------------------------------
## About

Kubernetes Engine is a managed, production-ready environment for deploying containerized applications. It brings our latest innovations in developer productivity, resource efficiency, automated operations, and open source flexibility to accelerate your time to market.

Google Cloud Platform are most eliable, efficient, and secured way to run Kubernetes clusters.

Preemptible VMs are Google Compute Engine VM instances that last a maximum of 24 hours and provide no availability guarantees. Preemptible VMs are priced lower than standard Compute Engine VMs and offer the same machine types and options.

You can use preemptible VMs in your Kubernetes Engine clusters or node pools to run batch or fault-tolerant jobs that are less sensitive to the ephemeral, non-guaranteed nature of preemptible VMs.

<br />

----------------------------------------------------------------------------------------------------

## Introduction

### Glossary
- Google Cloud Platform (GCP)
	- <https://console.cloud.google.com/>
- Google Kubernetes Engine (GKE)
	- References to "GKE" are made only when specific to the GKE implementation of Kubernetes (e.g. GKE preemptible instance type); else, the general "Kubernetes" descriptor is used.
- Kubernetes (a.k.a. "kube", "k8s")
	- References to "Kubernetes" are to the general Kubernetes concepts and principles and are not specific to the GKE implementation of Kubernetes.
	- The goal is to distinguish what would work on any Kubernetes cluster from what would work only on a GKE implementation.


### Requirements

- Google Cloud account. You can use free tier account
- Google Cloud Console compatible web browser
	-	Safari 10, Google Chrome 61, Firefox 56
- Terminal application
	- macOS Terminal.app, iTerm.app
	- Linux terminal
	- Putty
	- or, you can use Google Cloud Shell
- Preinstall gcloud command
- Developer editor (vim, nvim, Visual Studio Code, Atom, gEdit, etc.) with .yaml files support


### Before you begin

* For local development environmen to update all components run

```Shell
gcloud components update && gcloud components install kubectl
```

* Take the following steps to enable the Kubernetes Engine API:
	* Visit the Kubernetes Engine page in the Google Cloud Platform Console.
	* Create or select a project.
	* Wait for the API and related services to be enabled. This can take several minutes.
	* Make sure that billing is enabled for your project.

<br />

----------------------------------------------------------------------------------------------------

## Creating Google Cloud Engine cluster

### Setup environmen

Export SHELL ENV variables for $PROJECT_ID

```Shell
export PROJECT_ID=[PROJECT_ID]
```

Replace [PROJECT_ID] with your project ID.

### Configuring default settings for gcloud

#### Setting a default project

To set a default project, run the following command from Cloud Shell:

```Shell
gcloud config set project $PROJECT_ID
```

#### Setting a default compute zone

To set a default compute zone, run the following command:

```Shell
gcloud config set compute/zone us-central1-a
```

### Creating a Kubernetes Engine cluster

To create a cluster, run the following command:

```Shell
gcloud beta container clusters create meetup-prebake \
	--node-locations us-central1-a,us-central1-b,us-central1-c \
	--zone us-central1-a \
	--cluster-version 1.10.4-gke.0 \
	--machine-type g1-small \
	--num-nodes 2 \
	--enable-autoscaling \
	--enable-autoupgrade \
	--min-nodes 2 \
	--max-nodes 6 \
	--enable-cloud-monitoring \
	--enable-autorepair \
	--enable-stackdriver-kubernetes
```

### Get authentication credentials for the cluster

After creating your cluster, you need to get authentication credentials to interact with the cluster.
To authenticate for the cluster, run the following command:

```Shell
gcloud container clusters get-credentials meetup-prebake --zone us-central1-a
```

<br />

----------------------------------------------------------------------------------------------------

## Extend Google Cloud Kubernetes with preemptible Node Pool

### To add preemptible Node Pool for existed GKE cluster run comand:

```Shell
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

Note: Nodes in preemptible pool will have `cloud.google.com/gke-preeptible: true label.`

<br />

----------------------------------------------------------------------------------------------------

## Deploy Stress Workload

stress is a deliberately simple workload generator for POSIX systems. It imposes a configurable amount of CPU, memory, I/O, and disk stress on the system.


### To deploy Stress workload execute command:

```Shell
kubectl create -f stress-deployment.yaml
```

### Scale up the Stress deployment

```Shell
kubectl scale deployment stress --replicas=10
```

## Deploy Stress workload to PVM only nodes

```Shell
kubectl create -f stress-pvm-deployment.yaml
```

<br />

----------------------------------------------------------------------------------------------------

## Cluster Scaling with Kubernetes Operators

#### estafette-gke-preemptible-killer
**GitHub: <https://github.com/estafette/estafette-gke-preemptible-killer>**
> Kubernetes controller that ensures deletion of preemptible nodes in a GKE cluster is spread out to avoid the risk of all getting deleted at the same time after 24 hours.

- A small Kubernetes controller application which loops through a given pool of preemptible nodes.
- A task proactively kills a preemptible node (GCP instance type) before the regular 24-hour lifetime of a preemptible virtual machine.
- Kubernetes fills its spot with a new GKE preemtible node; the new node has a fresh 24-hour lifetime.
- This controller runs inside the cluster for ease of organization and access to the Kubernetes node instances that it inspects.


#### Deploy GKE Preemptible Killer

```Shell
export NAMESPACE=estafette
export APP_NAME=estafette-gke-preemptible-killer
export TEAM_NAME=tooling
export VERSION=1.0.35
export GO_PIPELINE_LABEL=1.0.35
export DRAIN_TIMEOUT=300
export INTERVAL=600
export CPU_REQUEST=10m
export MEMORY_REQUEST=16Mi
export CPU_LIMIT=50m
export MEMORY_LIMIT=128Mi
```

##### Setup RBAC

```Shell
curl https://raw.githubusercontent.com/estafette/estafette-gke-preemptible-killer/master/rbac.yaml | envsubst | kubectl apply -n ${NAMESPACE} -f -
```

##### Run application

```Shell
curl https://raw.githubusercontent.com/estafette/estafette-gke-preemptible-killer/master/kubernetes.yaml | envsubst | kubectl apply -n ${NAMESPACE} -f -
```
<br />

----------------------------------------------------------------------------------------------------

## Clean up

To avoid incurring charges to your Google Cloud Platform account for the resources used in this quickstart:

Delete your cluster by running gcloud container clusters delete:

```Shell
gcloud container clusters delete 
```
