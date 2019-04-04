# Kubes for Noobs



make a gcloud account

install gcloud cli tool https://cloud.google.com/sdk/docs/downloads-interactive

run 'kubectl' to see if you have kubectl installed. if not, run 

make sure 'brew update && brew upgrade kubernetes-cli'

create a project in gcloud

switch to the project by typing `gcloud config set project boondoggle1point0`

enable Kubernetes Engine API https://console.cloud.google.com/apis/

create a cluster to run the sample app kuar (from the O'reilly book kubernetes up and running) https://github.com/kubernetes-up-and-running/kuard

gcloud container clusters create kuar-cluster



