# Google Cloud Essentials

[toc]

## A Tour of Qwiklabs and Google Cloud

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

## Creating a Virtual Machine

- parameters

  - us-central1-f
  - n1-standard-2
  - 10 GB Debian 10
  - Allow HTTP

- SSH connect

- Install Nginx

  ```sh
  sudo su -
  apt-get update
  apt-get install nginx -y
  ps auwx | grep nginx
  ```

- Click  **External IP**  to see the web page

- Create a new instance and connect it with gcloud

  ```sh
  gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone us-central1-f
  ```

  ```sh
  gcloud compute instances create --help
  ```

  ```sh
  gcloud compute ssh gcelab2 --zone us-central1-f
  ```

## Getting Started with Cloud Shell and gcloud

- Configure your environment

  - If you want to attach a persistent disk to a virtual machine instance, both resources must be in the same zone
  - Assign a static IP address to an instance, the instance must be in the same region as the static IP address
  - default

  ```sh
  gcloud config get-value compute/zone
  gcloud config get-value compute/region
  gcloud compute project-info describe --project <your_project_ID>
  ```

- Set environment variables

  ```sh
  export PROJECT_ID=<your_project_ID>
  export ZONE=<your_zone>
  ```

- Create a virtual machine with the gcloud tool

  ```sh
  gcloud compute instances create --help
  gcloud compute instances create gcelab2 --machine-type n1-standard-2 --zone $ZONE
  ```

- Explore gcloud commands

  ```sh
  gcloud -h
  gcloud config --help
  gcloud config list
  gcloud config list --all
  gcloud components list
  ```

- Install a new component

  ```sh
  sudo apt-get install google-cloud-sdk
  gcloud beta interactive
  ```

- Connect to your VM instance with SSH

  ```sh
  gcloud compute ssh gcelab2 --zone $ZONE
  ```

- Use the Home directory

  ```sh
  cd $HOME
  vi ./.bashrc
  ```

## Kubernetes Engine: Qwik Start

- Set a default compute zone

  ```sh
  gcloud config set compute/zone us-central1-a
  ```

- Create a GKE cluster(A [cluster](https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-architecture) consists of at least one **cluster master** machine and multiple worker machines called **nodes**.)

  ```sh
  gcloud container clusters create my-cluster
  ```

- Get authentication credentials for the cluster

  ```sh
  gcloud container clusters get-credentials my-cluster
  ```

- Deploy an application to the cluster

  - GKE uses Kubernetes objects to create and manage your cluster's resources. Kubernetes provides the [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) object for deploying stateless applications like web servers. [Service](https://kubernetes.io/docs/concepts/services-networking/service/) objects define rules and load balancing for accessing your application from the internet.

  ```sh
  kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0
  kubectl expose deployment hello-server --type=LoadBalancer --port 8080
  kubectl get service
  ```

  - View the app - `http://[EXTERNAL-IP]:8080`

- Deleting the cluster

  ```sh
  gcloud container clusters delete my-cluster
  ```

## Set Up Network and HTTP Load Balancers

- Create an Network load balancer by following steps([ref](https://cloud.google.com/load-balancing/docs/network))

- Set the default region and zone for all resources

  ```sh
  gcloud config set compute/zone us-central1-a
  gcloud config set compute/region us-central1
  ```

- Create multiple web server instances

  ```sh
  gcloud compute instances create www1 \
    --image-family debian-9 \
    --image-project debian-cloud \
    --zone us-central1-a \
    --tags network-lb-tag \
    --metadata startup-script="#! /bin/bash
      sudo apt-get update
      sudo apt-get install apache2 -y
      sudo service apache2 restart
      echo '<!doctype html><html><body><h1>www1</h1></body></html>' | tee /var/www/html/index.html"
  
  gcloud compute instances create www2 \
    --image-family debian-9 \
    --image-project debian-cloud \
    --zone us-central1-a \
    --tags network-lb-tag \
    --metadata startup-script="#! /bin/bash
      sudo apt-get update
      sudo apt-get install apache2 -y
      sudo service apache2 restart
      echo '<!doctype html><html><body><h1>www2</h1></body></html>' | tee /var/www/html/index.html"
      
  gcloud compute instances create www3 \
    --image-family debian-9 \
    --image-project debian-cloud \
    --zone us-central1-a \
    --tags network-lb-tag \
    --metadata startup-script="#! /bin/bash
      sudo apt-get update
      sudo apt-get install apache2 -y
      sudo service apache2 restart
      echo '<!doctype html><html><body><h1>www3</h1></body></html>' | tee /var/www/html/index.html"
  ```

- Create a firewall rule to allow external traffic to the VM instances

  ```sh
  gcloud compute firewall-rules create www-firewall-network-lb \    --target-tags network-lb-tag --allow tcp:80
  ```

- Verify each instance is running with curl

  ```sh
  gcloud compute instances list
  curl http://[IP_ADDRESS]
  ```

- Create a static external IP address for your load balancer

  ```sh
  gcloud compute addresses create network-lb-ip-1 \
      --region us-central1
  ```

- Add a legacy HTTP health check resource

  ```sh
  gcloud compute http-health-checks create basic-check
  ```

- Create the target pool(in same region) and use the health check, which is required for the service to function

  ```sh
  gcloud compute target-pools create www-pool \
      --region us-central1 --http-health-check basic-check
  ```

- Add the instances to the pool:

  ```sh
  gcloud compute target-pools add-instances www-pool \
      --instances www1,www2,www3
  ```

- Add a forwarding rule

  ```sh
  gcloud compute forwarding-rules create www-rule \
      --region us-central1 \
      --ports 80 \
      --address network-lb-ip-1 \
      --target-pool www-pool
  ```

- Sending traffic to your instances

  ```sh
  gcloud compute forwarding-rules describe www-rule --region us-central1
  while true; do curl -m1 [IP_ADDRESS]; done
  ```

  ---

- Create an HTTP load balancer by following steps([ref](https://cloud.google.com/load-balancing/docs/https))

  - (HTTP(S) Load Balancing is implemented on Google Front End (GFE))
  - Requests are always routed to the instance group that is closest to the user

- Create the load balancer template

  ```sh
  gcloud compute instance-templates create lb-backend-template \
     --region=us-central1 \
     --network=default \
     --subnet=default \
     --tags=allow-health-check \
     --image-family=debian-9 \
     --image-project=debian-cloud \
     --metadata=startup-script='#! /bin/bash
       apt-get update
       apt-get install apache2 -y
       a2ensite default-ssl
       a2enmod ssl
       vm_hostname="$(curl -H "Metadata-Flavor:Google" \
       http://169.254.169.254/computeMetadata/v1/instance/name)"
       echo "Page served from: $vm_hostname" | \
       tee /var/www/html/index.html
       systemctl restart apache2'
  ```

- Create a managed instance group based on the template

  ```sh
  gcloud compute instance-groups managed create lb-backend-group \
      --template=lb-backend-template --size=2 --zone=us-central1-a
  ```

- Create the `fw-allow-health-check` firewall rule.

  ```sh
  gcloud compute firewall-rules create fw-allow-health-check \
      --network=default \
      --action=allow \
      --direction=ingress \
      --source-ranges=130.211.0.0/22,35.191.0.0/16 \
      --target-tags=allow-health-check \
      --rules=tcp:80
  ```

- Set up a global static external IP address

  ```sh
  gcloud compute addresses describe lb-ipv4-1 \
      --format="get(address)" \
      --global
  ```
  
- Create a healthcheck for the load balancer

  ```sh
  gcloud compute health-checks create http http-basic-check \
      --port 80
  ```

- Create a backend service

  ```sh
  gcloud compute backend-services create web-backend-service \
      --protocol=HTTP \
      --port-name=http \
      --health-checks=http-basic-check \
      --global
  ```

- Add your instance group as the backend to the backend service:

  ```sh
  gcloud compute backend-services add-backend web-backend-service \
      --instance-group=lb-backend-group \
      --instance-group-zone=us-central1-a \
      --global
  ```

- Create a URL map to route the incoming requests to the default backend service

  ```sh
  gcloud compute url-maps create web-map-http \
      --default-service web-backend-service
  ```

- Create a target HTTP proxy to route requests to your URL map

  ```sh
   gcloud compute target-http-proxies create http-lb-proxy \
      --url-map web-map-http
  ```

- Create a global forwarding rule to route incoming requests to the proxy

  ```sh
  gcloud compute forwarding-rules create http-content-rule \
      --address=lb-ipv4-1\
      --global \
      --target-http-proxy=http-lb-proxy \
      --ports=80
  ```

- Testing traffic sent to your instances

  - **Network services** > **Load balancing** > **web-map-http** >
  - Backend > Status = Ready
  - `http://[LOAD_BALANCER_IP_ADDRESS]/`
  - Your browser should render a page with content showing the name of the instance that served the page, along with its zone
