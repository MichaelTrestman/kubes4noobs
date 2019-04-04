# Kubes for Noobs



## Learning Objectives:

- understand what kubernetes is for 
- set up the tools on your machine and in the cloud Google's kubernetes engine
- create your first kubernetes cluster and deploy a sample app



## What is Kubernetes?



Kubernetes is a tool for orchestrating distributed computing application.

concepts:

- declarative configuration
  - understandibility
  - reproducibility
- containers
- pods
- self-healing
- scaling



## Getting a Handle on Kubernetes



#### Set up your gcloud shit

make a gcloud account

create a project in gcloud



install gcloud cli tool https://cloud.google.com/sdk/docs/downloads-interactive

switch to the project by typing `gcloud config set project boondoggle1point0`



#### Set up your kubectl shit

run `kubectl` to see if you have kubectl installed. if not, run 

make sure `brew update && brew upgrade kubernetes-cli`

enable Kubernetes Engine API https://console.cloud.google.com/apis/



## Create a cluster in the cloud



Create a cluster in your google cloud by running:

`$> gcloud container clusters create clusterfux`



The output should confirm that your cluster is up and running. This can take a few minutes.

```
Creating cluster clusterfux in us-west1-a... Cluster is being health-checked (master is

 healthy)...done.

Created [https://container.googleapis.com/v1/projects/boondoggle1point0/zones/us-west1-a/clusters/clusterfux].

To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-west1-a/clusterfux?project=boondoggle1point0

kubeconfig entry generated for clusterfux.

NAME        LOCATION    MASTER_VERSION  MASTER_IP      MACHINE_TYPE   NODE_VERSION   NUM_NODES  STATUS

clusterfux  us-west1-a  1.11.7-gke.12   35.197.16.123  n1-standard-1  1.11.7-gke.12  3          RUNNING

```



## Deploy a sample app




