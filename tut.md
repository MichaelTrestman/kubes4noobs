# Kubes for Noobs



## Learning Objectives:

- understand what kubernetes is for.
- set up the tools for using kubernetes, on your machine and in Google cloud.
- create your first kubernetes cluster and deploy a sample app.



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



### Set up your google cloud

1. Create a google cloud account for yourself, if you do not already have one, at https://cloud.google.com. Google offers a free trial account with plenty of credits.
2. Create a project in google cloud at https://console.cloud.google.com/projectcreate. Record the project name or keep the browser window open so you can find it easily.
3. Enable Kubernetes Engine API. Find it by searching at https://console.cloud.google.com/apis/ and then clicking 'enable'.
4. Install 'gcloud', the google cloud command line interface (CLI) tool https://cloud.google.com/sdk/docs/downloads-interactive.
5. Open iterm (on a mac) or your terminal of choice, and type `gcloud version` to ensure that gcloud is correctly installed
6. Configure the gcloud CLI to point to your new project by running   `gcloud config set project YOUR_PROJECT_NAME`
7. You can always check which project gcloud is targetting by running `gcloud config get-value project`, which should return the name of your project.



### Set up kubectl 

Kubectl is the command line interface (CLI) for Kubernetes. On a mac computer, the easiest way to manage kubectl is using homebrew.

1. Run `brew update` to make sure your local homebrew is up to date and knows about the latest version of kubectl.
2. Run `kubectl` to see if you have kubectl installed. 
   - If it is installed, you should see a summary of the manual page, including a list of top level commands. In this case, run `brew upgrade kubectl` to have homebrew bring your kubectl up to date.
   - If kubectl is not installed, your shell will tell you that the kubectl command cannot be found. In this case, run `brew install kubectl`. Now running the `kubectl` command should give you a summary of the manual page.



## Create a cluster in the cloud



1. If you prefer to create your cluster in a specific zone, you can target the zone by running `gcloud config set compute/zone PREFERRED_ZONE`, for example, you could use 'us-west1-a' as the value of `PREFERRED_ZONE`.

2. Create a cluster in your google cloud by running:

   `$> gcloud container clusters create YOUR_CLUSTER_NAME`

	he output should confirm that your cluster is up and running (this output below is for creation of a 		cluster called 'noobcluster' in a google cloud project called kubes4noobs). This can take a few minutes.

```
Creating cluster noobcluster in us-west1-a... Cluster is being health-checked (master is

 healthy)...done.

Created [https://container.googleapis.com/v1/projects/kubes4noobs/zones/us-west1-a/clusters/noobcluster].

To inspect the contents of your cluster, go to: https://console.cloud.google.com/kubernetes/workload_/gcloud/us-west1-a/noobcluster?project=kubes4noobs

kubeconfig entry generated for noobcluster.

NAME        LOCATION    MASTER_VERSION  MASTER_IP      MACHINE_TYPE   NODE_VERSION   NUM_NODES  STATUS

noobcluster  us-west1-a  1.11.7-gke.12   35.197.16.123  n1-standard-1  1.11.7-gke.12  3          RUNNING

```



## Deploy a sample app



Now we will deploy a simple app with a load balancer in front of a microservice that responds to requests by saying "hello".



We'll do this by running the `kubectl create deployment` command, pointing to a manifest that describes the a Kubernetes pod containing the app we want to deploy. The manifest will look like this:

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

You can also see that the `Events` section will show when your app scaled to seven instances as originally deployed, and then scaled down to three after you redeployed with the edited manifest:

```
:> kubectl describe deployments hello | grep -A4 Events
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  14m    deployment-controller  Scaled up replica set hello-7d7d777c6b to 7
  Normal  ScalingReplicaSet  4m36s  deployment-controller  Scaled down replica set hello-7d7d777c6b to 3
```



Currently, our application is not exposed to the internet. Do this, we will have Kuberneteds create a load balancer with a public IP address to which we can make requests via a web browser or the `curl` command.

