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



Now we will deploy a simple app with a load balancer in front of a microservice that responds to requests by saying "hello".



We'll do this by running the `kubectl create deployment` command, pointing to a manifest that describes the service we want to deploy. The manifest will look like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
spec:
  selector:
    matchLabels:
      app: hello
      tier: backend
      track: stable
  replicas: 7
  template:
    metadata:
      labels:
        app: hello
        tier: backend
        track: stable
    spec:
      containers:
        - name: hello
          image: "gcr.io/google-samples/hello-go-gke:1.0"
          ports:
            - name: http
              containerPort: 80
```



The manifest includes metadata and labels that the kubectle CLI uses to find the app in order to execute commands. It also includes the template metadata labels that define the labels for the *pod* that the app will exist in in our deployment, as well as the selector matchLabels that Kubernetes wil use to identify the pod in which to place the app. This is a bit confusing, but for now just be aware that these must match for Kubernetes to find a the pod in which to contain the app.

The `containers` section at the bottom defines a single container that will run our app. We give it a name and tell Kubernetes where to find the container image--in this case we will borrow it from the sample apps provided by Google on its public registry, gcr.io. We also tell it on which port to listen on requests, in this case port 80.

Create the deployment by telling Kubernetes to apply the manifest at the given file path. You can either a) copy the manifest above into a file on your local file system, and use the path to the manifest on your local file system, or b) use the following path where it is hosted by k8s.io as an example: https://k8s.io/examples/service/access/hello.yaml

`kubectl apply -f PATH_TO_MANIFEST`

You should see confirmation that the deployment has been created `deployment.apps/hello created`

If you ask Kubernetes to list your deployments, you should see it listed:

```
:> kubectl get deployments
NAME    DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello   7         7         7            7           1m
```



You can get detailed information about the deployment asking Kubernetes to describe it, using the name we defined in the manifest, "hello":

 `kubectl describe deployments hello`

The manifest we used to create the deployment asks for 7 replicas of the pod containing our application. This is more than we need for this simple example, so let's scale it back by editing the manifest.

Run `kubectl edit deployments hello` to edit the manifest in your default text editor. Under the `spec` key, edit the value of the `replicas` key to be 3, rather than seven. Save and close the file, and Kubernetes will apply your changes.

If you run  `kubectl describe deployments hello` again, you should see that there are now only 3 replicas. 

```
:> kubectl describe deployments hello | grep Replicas:
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
```

You can also see that the `Events` section will show when your app scaled to 7 instances as originally deployed, and then scaled down to three after you redeployed with the edited manifest:

```
:> kubectl describe deployments hello | grep -A4 Events
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  14m    deployment-controller  Scaled up replica set hello-7d7d777c6b to 7
  Normal  ScalingReplicaSet  4m36s  deployment-controller  Scaled down replica set hello-7d7d777c6b to 3
```



Currently, our application is not exposed to the internet. Do this, we will have Kuberneteds create a load balancer with a public IP address to which we can make requests via a web browser or the `curl` command.





