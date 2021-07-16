

# Qwiklabs

## Google Cloud Essentials

### A Tour of Qwiklabs and Google Cloud

1. Google Cloud 7 Services([ref](https://cloud.google.com/docs/overview/cloud-platform-services#top_of_page))
   - Compute
   - Storage
   - Networking
   - Cloud Operations
   - Tools
   - Big Data
   - AI

2. IAM & Admin

   - Role & Permissions

3. APIs & Services

   - APIs & Services > Library

   - [Google APIs Explorer](https://developers.google.com/apis-explorer/#p/)

   - [APIs Explorer: Qwik Start](https://google.qwiklabs.com/catalog_lab/1241)

4. Cloud Shell

   - gcloud
   - Unix CLI

### Creating a Virtual Machine

- parameters

  - us-central1-f
  - n1-standard-2
  - 10 GB Debian 10
  - Allow HTTP

- SSH connect

- Install Nginx

  ```
  sudo su -
  apt-get update
  apt-get install nginx -y
  ps auwx | grep nginx
  ```

- Click  **External IP**  to see the web page

- Create a new instance and connect it with gcloud 

  ```
  gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone us-central1-f
  ```

  ```
  gcloud compute instances create --help
  ```

  ```
  gcloud compute ssh gcelab2 --zone us-central1-f
  ```

### Getting Started with Cloud Shell and gcloud

- Configure your environment

  -  If you want to attach a persistent disk to a virtual machine instance, both resources must be in the same zone
  -  Assign a static IP address to an instance, the instance must be in the same region as the static IP address
  -  default

  ```
  gcloud config get-value compute/zone
  gcloud config get-value compute/region
  gcloud compute project-info describe --project <your_project_ID>
  ```

- Set environment variables

  ```
  export PROJECT_ID=<your_project_ID>
  export ZONE=<your_zone>
  ```

- Create a virtual machine with the gcloud tool

  ```
  gcloud compute instances create --help
  gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone $ZONE
  ```

- Explore gcloud commands

  ```
  gcloud -h
  gcloud config --help
  gcloud config list
  gcloud config list --all
  gcloud components list
  ```

- Install a new component

  ```
  sudo apt-get install google-cloud-sdkgcloud beta interactive
  ```

- Connect to your VM instance with SSH

  ```
  gcloud compute ssh gcelab2 --zone $ZONE
  ```

- Use the Home directory

  ```
  cd $HOMEvi ./.bashrc
  ```

### Kubernetes Engine: Qwik Start

- Set a default compute zone

  ```
  gcloud config set compute/zone us-central1-a
  ```

- Create a GKE cluster(A [cluster](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-architecture) consists of at least one **cluster master** machine and multiple worker machines called **nodes**.)

  ```
  gcloud container clusters create my-cluster
  ```

- Get authentication credentials for the cluster

  ```
  gcloud container clusters get-credentials my-cluster
  ```

- Deploy an application to the cluster

  - GKE uses Kubernetes objects to create and manage your cluster's resources. Kubernetes provides the [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) object for deploying stateless applications like web servers. [Service](https://kubernetes.io/docs/concepts/services-networking/service/) objects define rules and load balancing for accessing your application from the internet.

  ```
  kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0kubectl expose deployment hello-server --type=LoadBalancer --port 8080kubectl get service
  ```

  - View the app - http://[EXTERNAL-IP]:8080

- Deleting the cluster

  ```
  gcloud container clusters delete my-cluster
  ```
