# Kubernetes meet-up: cost-effective Kubernetes DEMO
_How-to decrease cost with Preemptible Compute Engine Machines_

----------------------------------------------------------------------------------------------------
## About

Kubernetes Engine is a managed, production-ready environment for deploying containerized applications. It brings our latest innovations in developer productivity, resource efficiency, automated operations, and open source flexibility to accelerate your time to market.

Google Cloud Platform are most eliable, efficient, and secured way to run Kubernetes clusters.

Preemptible VMs are Google Compute Engine VM instances that last a maximum of 24 hours and provide no availability guarantees. Preemptible VMs are priced lower than standard Compute Engine VMs and offer the same machine types and options.

You can use preemptible VMs in your Kubernetes Engine clusters or node pools to run batch or fault-tolerant jobs that are less sensitive to the ephemeral, non-guaranteed nature of preemptible VMs.

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

----------------------------------------------------------------------------------------------------

## Creating Google Cloud Engine cluster

### Configuring default settings for gcloud

#### Setting a default project

To set a default project, run the following command from Cloud Shell:

```Shell
gcloud config set project [PROJECT_ID]
```

Replace [PROJECT_ID] with your project ID.

#### Setting a default compute zone

To set a default compute zone, run the following command:

```Shell
gcloud config set compute/zone [COMPUTE_ZONE]
```

where [COMPUTE_ZONE] is the desired geographical compute zone, such as us-west1-a.


### Creating a Kubernetes Engine cluster

To create a cluster, run the following command:

```Shell
gcloud container node-pools create preemtible-pool \
	--cluster [CLUSTER_NAME] \
	--zone [CLUSTER_ZONE] \
	--enable-autoupgrade \
	--num-nodes 1 --machine-type n1-standard-8 \
	--enable-autoscaling --min-nodes=0 --max-nodes=6
```

where [CLUSTER_NAME] is the name you choose for the cluster.

### Get authentication credentials for the cluster

After creating your cluster, you need to get authentication credentials to interact with the cluster.
To authenticate for the cluster, run the following command:

```Shell
gcloud container clusters get-credentials [CLUSTER_NAME]
```

----------------------------------------------------------------------------------------------------

## Extend Google Cloud Kubernetes with preemptible Node Pool

### To add preemptible Node Pool for existed GKE cluster run comand:

```Shell
gcloud container node-pools create preemtible-pool \
	--cluster $CLUSTER_NAME \
	--zone $CLUSTER_ZONE \
	--scopes cloud-platform \
	--enable-autoupgrade \
	--preemptible \
	--num-nodes 1 --machine-type n1-standard-8 \
	--enable-autoscaling --min-nodes=0 --max-nodes=6
```

where [CLUSTER_NAME] is the name you choose for the cluster;
where [COMPUTE_ZONE] is the desired geographical compute zone, such as us-west1-a.


Note: Nodes in preemptible pool will have `cloud.google.com/gke-preeptible: true label.`

----------------------------------------------------------------------------------------------------

## Cluster Scaling 

#### estafette-gke-preemptible-killer
**GitHub: <https://github.com/estafette/estafette-gke-preemptible-killer>**
> Kubernetes controller that ensures deletion of preemptible nodes in a GKE cluster is spread out to avoid the risk of all getting deleted at the same time after 24 hours.

- A small Kubernetes controller application which loops through a given pool of preemptible nodes.
- A task proactively kills a preemptible node (GCP instance type) before the regular 24-hour lifetime of a preemptible virtual machine.
- Kubernetes fills its spot with a new GKE preemtible node; the new node has a fresh 24-hour lifetime.
- This controller runs inside the cluster for ease of organization and access to the Kubernetes node instances that it inspects.


----------------------------------------------------------------------------------------------------

## Clean up

To avoid incurring charges to your Google Cloud Platform account for the resources used in this quickstart:

Delete your cluster by running gcloud container clusters delete:

`gcloud container clusters delete [CLUSTER_NAME]`

----------------------------------------------------------------------------------------------------

## Inventory of this repository

## Kubernetes cluster: app stack
_Components and configurations for assembling a base Kubernetes cluster on GKE._
