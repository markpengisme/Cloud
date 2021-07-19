

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

### Set Up Network and HTTP Load Balancers

- Create an Network load balancer by following steps([ref](https://cloud.google.com/load-balancing/docs/network))

- Set the default region and zone for all resources

  ```
  gcloud config set compute/zone us-central1-agcloud config set compute/region us-central1
  ```

- Create multiple web server instances

  ```
  gcloud compute instances create www1 \  --image-family debian-9 \  --image-project debian-cloud \  --zone us-central1-a \  --tags network-lb-tag \  --metadata startup-script="#! /bin/bash    sudo apt-get update    sudo apt-get install apache2 -y    sudo service apache2 restart    echo '<!doctype html><html><body><h1>www1</h1></body></html>' | tee /var/www/html/index.html"gcloud compute instances create www2 \  --image-family debian-9 \  --image-project debian-cloud \  --zone us-central1-a \  --tags network-lb-tag \  --metadata startup-script="#! /bin/bash    sudo apt-get update    sudo apt-get install apache2 -y    sudo service apache2 restart    echo '<!doctype html><html><body><h1>www2</h1></body></html>' | tee /var/www/html/index.html"gcloud compute instances create www3 \  --image-family debian-9 \  --image-project debian-cloud \  --zone us-central1-a \  --tags network-lb-tag \  --metadata startup-script="#! /bin/bash    sudo apt-get update    sudo apt-get install apache2 -y    sudo service apache2 restart    echo '<!doctype html><html><body><h1>www3</h1></body></html>' | tee /var/www/html/index.html"
  ```

- Create a firewall rule to allow external traffic to the VM instances

  ```
  gcloud compute firewall-rules create www-firewall-network-lb \    --target-tags network-lb-tag --allow tcp:80
  ```

- Verify each instance is running with curl

  ```
  gcloud compute instances listcurl http://[IP_ADDRESS]
  ```

- Create a static external IP address for your load balancer

  ```
  gcloud compute addresses create network-lb-ip-1 \ --region us-central1
  ```

- Add a legacy HTTP health check resource

  ```
  gcloud compute http-health-checks create basic-check
  ```

- Create the target pool(in same region) and use the health check, which is required for the service to function

  ```
  gcloud compute target-pools create www-pool \    --region us-central1 --http-health-check basic-check
  ```

- Add the instances to the pool:

  ```
  gcloud compute target-pools add-instances www-pool \    --instances www1,www2,www3
  ```

- Add a forwarding rule

  ```
  gcloud compute forwarding-rules create www-rule \    --region us-central1 \    --ports 80 \    --address network-lb-ip-1 \    --target-pool www-pool
  ```

- Sending traffic to your instances

  ```
  gcloud compute forwarding-rules describe www-rule --region us-central1while true; do curl -m1 [IP_ADDRESS]; done
  ```

  ---

- Create an HTTP load balancer by following steps([ref](https://cloud.google.com/load-balancing/docs/https))

  - (HTTP(S) Load Balancing is implemented on Google Front End (GFE))
  - Requests are always routed to the instance group that is closest to the user

- Create the load balancer template

  ```
  gcloud compute instance-templates create lb-backend-template \   --region=us-central1 \   --network=default \   --subnet=default \   --tags=allow-health-check \   --image-family=debian-9 \   --image-project=debian-cloud \   --metadata=startup-script='#! /bin/bash     apt-get update     apt-get install apache2 -y     a2ensite default-ssl     a2enmod ssl     vm_hostname="$(curl -H "Metadata-Flavor:Google" \     http://169.254.169.254/computeMetadata/v1/instance/name)"     echo "Page served from: $vm_hostname" | \     tee /var/www/html/index.html     systemctl restart apache2'
  ```

- Create a managed instance group based on the template

  ```
  gcloud compute instance-groups managed create lb-backend-group \   --template=lb-backend-template --size=2 --zone=us-central1-a
  ```

- Create the `fw-allow-health-check` firewall rule.

  ```
  gcloud compute firewall-rules create fw-allow-health-check \    --network=default \    --action=allow \    --direction=ingress \    --source-ranges=130.211.0.0/22,35.191.0.0/16 \    --target-tags=allow-health-check \    --rules=tcp:80
  ```

- Set up a global static external IP address

  ```
  gcloud compute addresses create lb-ipv4-1 \    --ip-version=IPV4 \    --globalgcloud compute addresses describe lb-ipv4-1 \    --format="get(address)" \    --global
  ```

- Create a healthcheck for the load balancer

  ```
  gcloud compute health-checks create http http-basic-check \	--port 80
  ```

- Create a backend service

  ```
  gcloud compute backend-services create web-backend-service \    --protocol=HTTP \    --port-name=http \    --health-checks=http-basic-check \    --global
  ```

- Add your instance group as the backend to the backend service:

  ```
  gcloud compute backend-services add-backend web-backend-service \    --instance-group=lb-backend-group \    --instance-group-zone=us-central1-a \    --global
  ```

- Create a URL map to route the incoming requests to the default backend service

  ```
  gcloud compute url-maps create web-map-http \	--default-service web-backend-service
  ```

- Create a target HTTP proxy to route requests to your URL map

  ```
   gcloud compute target-http-proxies create http-lb-proxy \	--url-map web-map-http
  ```

- Create a global forwarding rule to route incoming requests to the proxy

  ```
  gcloud compute forwarding-rules create http-content-rule \    --address=lb-ipv4-1\    --global \    --target-http-proxy=http-lb-proxy \    --ports=80
  ```

- Testing traffic sent to your instances

  - **Network services** > **Load balancing **>**web-map-http**>
  - Backend > Status = Ready
  - http://[LOAD_BALANCER_IP_ADDRESS]/
  - Your browser should render a page with content showing the name of the instance that served the page, along with its zone 

## Build a Website on Google Cloud

### Deploy Your Website on Cloud Run

- Activate Cloud Shell

  ```
  gcloud auth list
  gcloud config list project
  ```

- Clone Source Repository

  - clone > start > Preview on port 8080

  ```
  git clone https://github.com/googlecodelabs/monolith-to-microservices.git
  cd ~/monolith-to-microservices
  ./setup.sh
  
  cd ~/monolith-to-microservices/monolith
  npm start
  ```

- Create Docker Container with Cloud Build

  - build > submit > view[`Navigation > Cloud Build > History`]

  ```
  gcloud services enable cloudbuild.googleapis.com
  gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 .
  ```

- Deploy Container To Cloud Run

  - There are two approaches for deploying to Cloud Run:

    - Managed Cloud Run(this lab)

    - Cloud Run on GKE([ref](https://cloud.google.com/run/docs/gke/setup))

  ```
  gcloud services enable run.googleapis.com
  gcloud run deploy --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 --platform managed
  ```

- Verify deployment

  - `gcloud run services list`
  - OR
  - [`Navigation > Cloud Run`]

- Create new revision with lower concurrency(default is 80)

  ```
  gcloud run deploy --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 --platform managed --concurrency 1
  ```

  - [`Navigation > Cloud Run > monolith > Revisions > monolith-00002`] Concurrency=1

  ```
  gcloud run deploy --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 --platform managed --concurrency 80
  ```

- Make Changes To The Website

  ```sh
  ## update
  cd ~/monolith-to-microservices/react-app/src/pages/Home
  mv index.js.new index.js
  cat ~/monolith-to-microservices/react-app/src/pages/Home/index.js
  cd ~/monolith-to-microservices/react-app
  
  ## build react app
  npm run build:monolith
  cd ~/monolith-to-microservices/monolith
  npm start
  
  ## build & submit docker image
  gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0 .
  ```

- Update website with zero downtime

  ```
  gcloud run deploy --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0 --platform managed
  ```

- Verify Deployment

  ```SH
  gcloud run services describe monolith --platform managed
  
  ## see IP
  gcloud beta run services list
  ```

- Cleanup

  ```sh
  # Delete the container image for version 1.0.0 of our monolith
  gcloud container images delete gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 --quiet
  
  # Delete the container image for version 2.0.0 of our monolith
  gcloud container images delete gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0 --quiet
  
  # The following command will take all source archives from all builds and delete them from cloud storage
  gcloud builds list | awk 'NR > 1 {print $4}' | while read line; do gsutil rm $line; done
  
  # Delete Cloud Run service(choose region)
  gcloud beta run services delete monolith --platform managed
  
  # Check
  gcloud beta run services list
  ```

### Hosting a Web App on Google Cloud Using Compute Engine

:exclamation:

- Environment Setup

  ```
  gcloud auth list
  gcloud config list project
  gcloud config set compute/zone us-central1-f
  gcloud services enable compute.googleapis.com
  ```

-  Create Cloud Storage bucket

  - `$DEVSHELL_PROJECT_ID`: help for unique

  ```
  gsutil mb gs://fancy-store-$DEVSHELL_PROJECT_ID
  ```

- Clone source repository

  ```
  git clone https://github.com/googlecodelabs/monolith-to-microservices.git
  cd ~/monolith-to-microservices
  ./setup.sh
  
  cd microservices
  npm start
  ```

  - Preview on port 8080

- Create Compute Engine instances

  1. Create Startup Script

     - **Open Editor** > **File** > **New File **>`monolith-to-microservices/startup-script.sh`
     - command+option+shift+v = paste & match style
     - replace  [DEVSHELL_PROJECT_ID] to yours
     - check  "End of Line Sequence" is set to "LF"

     ```sh
     #!/bin/bash
     
     
     # Install logging monitor. The monitor will automatically pick up logs sent to
     # syslog.
     curl -s "https://storage.googleapis.com/signals-agents/logging/google-fluentd-install.sh" | bash
     service google-fluentd restart &
     
     # Install dependencies from apt
     apt-get update
     apt-get install -yq ca-certificates git build-essential supervisor psmisc
     
     # Install nodejs
     mkdir /opt/nodejs
     curl https://nodejs.org/dist/v8.12.0/node-v8.12.0-linux-x64.tar.gz | tar xvzf - -C /opt/nodejs --strip-components=1
     ln -s /opt/nodejs/bin/node /usr/bin/node
     ln -s /opt/nodejs/bin/npm /usr/bin/npm
     
     # Get the application source code from the Google Cloud Storage bucket.
     mkdir /fancy-store
     gsutil -m cp -r gs://fancy-store-[DEVSHELL_PROJECT_ID]/monolith-to-microservices/microservices/* /fancy-store/
     
     # Install app dependencies.
     cd /fancy-store/
     npm install
     
     # Create a nodeapp user. The application will run as this user.
     useradd -m -d /home/nodeapp nodeapp
     chown -R nodeapp:nodeapp /opt/app
     
     # Configure supervisor to run the node app.
     cat >/etc/supervisor/conf.d/node-app.conf << EOF
     [program:nodeapp]
     directory=/fancy-store
     command=npm start
     autostart=true
     autorestart=true
     user=nodeapp
     environment=HOME="/home/nodeapp",USER="nodeapp",NODE_ENV="production"
     stdout_logfile=syslog
     stderr_logfile=syslog
     EOF
     
     supervisorctl reread
     supervisorctl update
     ```

     ```
     gsutil cp ~/monolith-to-microservices/startup-script.sh gs://fancy-store-$DEVSHELL_PROJECT_ID
     ```

     - It will now be accessible at: `https://storage.googleapis.com/[BUCKET_NAME]/startup-script.sh`.

  2. Copy code into Cloud Storage bucket

     ```
     cd ~
     rm -rf monolith-to-microservices/*/node_modules
     gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
     ```

  3. Deploy backend instance

     ```
     gcloud compute instances create backend \
         --machine-type=n1-standard-1 \
         --tags=backend \
        --metadata=startup-script-url=https://storage.googleapis.com/fancy-store-$DEVSHELL_PROJECT_ID/startup-script.sh
     ```

  4. Configure connection to backend

     - `gcloud compute instances list` > Copy the External IP

     - **Open Editor** > **File** >**View** > **Toggle Hidden Files**

     - `monolith-to-microservices/react-app/.env` replace localhost to External IP

     - Rebuild react-app > copt to bucket

       ```
       cd ~/monolith-to-microservices/react-app
       npm install && npm run-script build
       cd ~
       rm -rf monolith-to-microservices/*/node_modules
       gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
       ```

  5. Deploy frontend instance

     ```
     gcloud compute instances create frontend \
         --machine-type=n1-standard-1 \
         --tags=frontend \
         --metadata=startup-script-url=https://storage.googleapis.com/fancy-store-$DEVSHELL_PROJECT_ID/startup-script.sh
     ```

     > all microservices run on both the frontend and backend in this sample

  6. Configure Network

     ```sh
     gcloud compute firewall-rules create fw-fe \
         --allow tcp:8080 \
         --target-tags=frontend
         
     gcloud compute firewall-rules create fw-be \
         --allow tcp:8081-8082 \
         --target-tags=backend
         
     ## use `gcloud compute instances list` to replace frontend external ip
     watch -n 2 curl http://[FRONTEND_ADDRESS]:8080
     ```

     - open browser to `http://[FRONTEND_ADDRESS]:8080`

- Create Managed Instance Groups(MIG)

  - To allow the application to scale
  - manage as a single entity in a single zone. 
  - provide autohealing, load balancing, autoscaling, and rolling updates

  - Create Instance Template from Source Instance

    ```
    gcloud compute instances stop frontend
    gcloud compute instances stop backend
    
    gcloud compute instance-templates create fancy-fe \
        --source-instance=frontend
    gcloud compute instance-templates create fancy-be \
        --source-instance=backend
        
    gcloud compute instance-templates list
    
    # delete backend vm to save resource
    gcloud compute instances delete backend
    ```

  - Create managed instance group

    ```
    gcloud compute instance-groups managed create fancy-fe-mig \
        --base-instance-name fancy-fe \
        --size 2 \
        --template fancy-fe
        
    gcloud compute instance-groups managed create fancy-be-mig \
        --base-instance-name fancy-be \
        --size 2 \
        --template fancy-be
        
    gcloud compute instance-groups set-named-ports fancy-fe-mig \
        --named-ports frontend:8080
        
    gcloud compute instance-groups set-named-ports fancy-be-mig \
        --named-ports orders:8081,products:8082
    ```

    - These managed instance groups will use the instance templates and are configured for two instances each within each group to start

  - Configure autohealing using health check

    ```sh
    ## create health check(unhealthy 3 times)
    gcloud compute health-checks create http fancy-fe-hc \
        --port 8080 \
        --check-interval 30s \
        --healthy-threshold 1 \
        --timeout 10s \
        --unhealthy-threshold 3
    gcloud compute health-checks create http fancy-be-hc \
        --port 8081 \
        --request-path=/api/orders \
        --check-interval 30s \
        --healthy-threshold 1 \
        --timeout 10s \
        --unhealthy-threshold 3
    
    ## firewall rule
    gcloud compute firewall-rules create allow-health-check \
        --allow tcp:8080-8081 \
        --source-ranges 130.211.0.0/22,35.191.0.0/16 \
        --network default
    
    ## apply health check
    gcloud compute instance-groups managed update fancy-fe-mig \
        --health-check fancy-fe-hc \
        --initial-delay 300
        
    gcloud compute instance-groups managed update fancy-be-mig \
        --health-check fancy-be-hc \
        --initial-delay 300
    ```

- Create Load Balancers

  - Create HTTP(S) Load Balancer

    - using mappings to send traffic to the proper backend services based on pathing rules

    ```sh
    ## create load balancer health check
    gcloud compute http-health-checks create fancy-fe-frontend-hc \
      --request-path / \
      --port 8080
    gcloud compute http-health-checks create fancy-be-orders-hc \
      --request-path /api/orders \
      --port 8081
    gcloud compute http-health-checks create fancy-be-products-hc \
      --request-path /api/products \
      --port 8082
      
    ## create backend services target for load-balanced traffic
    gcloud compute backend-services create fancy-fe-frontend \
      --http-health-checks fancy-fe-frontend-hc \
      --port-name frontend \
      --global
    gcloud compute backend-services create fancy-be-orders \
      --http-health-checks fancy-be-orders-hc \
      --port-name orders \
      --global
    gcloud compute backend-services create fancy-be-products \
      --http-health-checks fancy-be-products-hc \
      --port-name products \
      --global
      
    ## add backend services
    gcloud compute backend-services add-backend fancy-fe-frontend \
      --instance-group fancy-fe-mig \
      --instance-group-zone us-central1-f \
      --global
    gcloud compute backend-services add-backend fancy-be-orders \
      --instance-group fancy-be-mig \
      --instance-group-zone us-central1-f \
      --global
    gcloud compute backend-services add-backend fancy-be-products \
      --instance-group fancy-be-mig \
      --instance-group-zone us-central1-f \
      --global
      
    ## create url map
    gcloud compute url-maps create fancy-map \
      --default-service fancy-fe-frontend
      
    ## create path matcher allow orders and products route to their respective services
    gcloud compute url-maps add-path-matcher fancy-map \
       --default-service fancy-fe-frontend \
       --path-matcher-name orders \
       --path-rules "/api/orders=fancy-be-orders,/api/products=fancy-be-products"
       
    ## create proxy which ties to url map
    gcloud compute target-http-proxies create fancy-proxy \
      --url-map fancy-map
      
    ## create global forwarding rule which ties a IP&Port to the proxy
    gcloud compute forwarding-rules create fancy-http-rule \
      --global \
      --target-http-proxy fancy-proxy \
      --ports 80
    ```

- Update Configuration(backend IP)

  ```
  cd ~/monolith-to-microservices/react-app/
  gcloud compute forwarding-rules list --global
  ```

  - `monolith-to-microservices/react-app/.env` replace to load balancer external IP

  ```
  cd ~/monolith-to-microservices/react-app
  npm install && npm run-script build
  
  cd ~
  rm -rf monolith-to-microservices/*/node_modules
  gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
  ```

- Update the frontend instances by rolling restart command

  ```
  gcloud compute instance-groups managed rolling-action replace fancy-fe-mig --max-unavailable 100%
  ```

- Test the website

  ```sh
  ## frontend running and healthy(need wait for it)
  watch -n 2 gcloud compute instance-groups list-instances fancy-fe-mig
  watch -n 2 gcloud compute backend-services get-health fancy-fe-frontend --global
  ```

- Scaling Compute Engine 

  - Automatically Resize by Utilization(`>60`->add;` <60`->remove)

    ```
    gcloud compute instance-groups managed set-autoscaling \
      fancy-fe-mig \
      --max-num-replicas 2 \
      --target-load-balancing-utilization 0.60
    gcloud compute instance-groups managed set-autoscaling \
      fancy-be-mig \
      --max-num-replicas 2 \
      --target-load-balancing-utilization 0.60
    ```

  - Enable Content Delivery Network to provide caching for the frontend.(GFE provide)

    ```
    gcloud compute backend-services update fancy-fe-frontend \
        --enable-cdn --global
    ```

- Update the website

  - Updating Instance Template

      - Update the `frontend` instance, which acts as the basis for the instance template. During the update, you will put a file on the updated version of the instance template's image, then update the instance template, roll out the new template, and then confirm the file exists on the managed instance group instances.

      ```sh
      ## update machine type of instance template
      gcloud compute instances set-machine-type frontend --machine-type custom-4-3840
      ## create new instance template
      gcloud compute instance-templates create fancy-fe-new \
          --source-instance=frontend \
          --source-instance-zone=us-central1-f
      ## roll out
      gcloud compute instance-groups managed rolling-action start-update fancy-fe-mig \
          --version template=fancy-fe-new
      ```

      ```sh
      ## Wait for Running|None|fancy-fe-new
      watch -n 2 gcloud compute instance-groups managed list-instances fancy-fe-mig
      gcloud compute instances describe [VM_NAME] | grep machineType
      ```

  - Make changes to the website

    - **Scenario:** Your marketing team has asked you to change the homepage for your site. They think it should be more informative of who your company is and what you actually sell.

    ```sh
    ## update
    cd ~/monolith-to-microservices/react-app/src/pages/Home
    mv index.js.new index.js
    cat ~/monolith-to-microservices/react-app/src/pages/Home/index.js
    cd ~/monolith-to-microservices/react-app
    npm install && npm run-script build
    cd ~
    rm -rf monolith-to-microservices/*/node_modules
    gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
    
    ## restart
    gcloud compute instance-groups managed rolling-action replace fancy-fe-mig --max-unavailable=100%
    
    ## wait 30s, comfirm Healthy
    watch -n 2 gcloud compute instance-groups list-instances fancy-fe-mig
    watch -n 2 gcloud compute backend-services get-health fancy-fe-frontend --global
    
    gcloud compute forwarding-rules list --global
    ```

    - http://[LB_IP]

  - Simulate Failure: In order to confirm the health check works, log in to an instance and stop the services.

    ```sh
    gcloud compute instance-groups list-instances fancy-fe-mig
    gcloud compute ssh [INSTANCE_NAME]
    sudo supervisorctl stop nodeapp; sudo killall node
    exit
    
    ## monitor repair operations or view at `Navigation menu > Compute Engine > Instance groups`
    watch -n 2 gcloud compute operations list \
    --filter='operationType~compute.instances.repair.*'
    ```

    

