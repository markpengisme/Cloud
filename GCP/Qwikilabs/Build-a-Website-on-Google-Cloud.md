# Build a Website on Google Cloud

[toc]

## Deploy Your Website on Cloud Run

- Activate Cloud Shell

  ```sh
  gcloud auth list
  gcloud config list project
  ```

- Clone Source Repository

  - clone > start > Preview on port 8080

  ```sh
  git clone https://github.com/googlecodelabs/monolith-to-microservices.git
  cd ~/monolith-to-microservices
  ./setup.sh
  
  cd ~/monolith-to-microservices/monolith
  npm start
  ```

- Create Docker Container with Cloud Build

  - build > submit > view[`Navigation > Cloud Build > History`]

  ```sh
  gcloud services enable cloudbuild.googleapis.com
  gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 .
  ```

- Deploy Container To Cloud Run

  - There are two approaches for deploying to Cloud Run:

    - Managed Cloud Run(this lab)

    - Cloud Run on GKE([ref](https://cloud.google.com/run/docs/gke/setup))

  ```sh
  gcloud services enable run.googleapis.com
  gcloud run deploy --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 --platform managed
  ```

- Verify deployment

  - `gcloud run services list`
  - OR
  - [`Navigation > Cloud Run`]

- Create new revision with lower concurrency(default is 80)

  ```sh
  gcloud run deploy --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 --platform managed --concurrency 1
  ```

  - [`Navigation > Cloud Run > monolith > Revisions > monolith-00002`] Concurrency=1

  ```sh
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

  ```sh
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

## Hosting a Web App on Google Cloud Using Compute Engine

:exclamation:

- Environment Setup

  ```sh
  gcloud auth list
  gcloud config list project
  gcloud config set compute/zone us-central1-f
  gcloud services enable compute.googleapis.com
  ```

- Create Cloud Storage bucket

  - `$DEVSHELL_PROJECT_ID`: help for unique

  ```sh
  gsutil mb gs://fancy-store-$DEVSHELL_PROJECT_ID
  ```

- Clone source repository

  ```sh
  git clone https://github.com/googlecodelabs/monolith-to-microservices.git
  cd ~/monolith-to-microservices
  ./setup.sh
  
  cd microservices
  npm start
  ```

  - Preview on port 8080

- Create Compute Engine instances

  1. Create Startup Script

     - **Open Editor** > **File** > **New File** > `monolith-to-microservices/startup-script.sh`
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

     ```sh
     gsutil cp ~/monolith-to-microservices/startup-script.sh gs://fancy-store-$DEVSHELL_PROJECT_ID
     ```

     - It will now be accessible at: `https://storage.googleapis.com/[BUCKET_NAME]/startup-script.sh`.

  2. Copy code into Cloud Storage bucket
  
     ```sh
     cd ~
     rm -rf monolith-to-microservices/*/node_modules
     gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
     ```
  
  3. Deploy backend instance
  
     ```sh
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
  
       ```sh
       cd ~/monolith-to-microservices/react-app
       npm install && npm run-script build
       cd ~
       rm -rf monolith-to-microservices/*/node_modules
       gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
       ```
  
  5. Deploy frontend instance
  
     ```sh
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

    ```sh
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

    ```sh
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

  ```sh
  cd ~/monolith-to-microservices/react-app/
  gcloud compute forwarding-rules list --global
  ```

  - `monolith-to-microservices/react-app/.env` replace to load balancer external IP

  ```sh
  cd ~/monolith-to-microservices/react-app
  npm install && npm run-script build
  
  cd ~
  rm -rf monolith-to-microservices/*/node_modules
  gsutil -m cp -r monolith-to-microservices gs://fancy-store-$DEVSHELL_PROJECT_ID/
  ```

- Update the frontend instances by rolling restart command

  ```sh
  gcloud compute instance-groups managed rolling-action replace fancy-fe-mig --max-unavailable 100%
  ```

- Test the website

  ```sh
  ## frontend running and healthy(need wait for it)
  watch -n 2 gcloud compute instance-groups list-instances fancy-fe-mig
  watch -n 2 gcloud compute backend-services get-health fancy-fe-frontend --global
  ```

- Scaling Compute Engine

  - Automatically Resize by Utilization(`>60`->add; `<60`->remove)

    ```sh
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

    ```sh
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

    - `http://[LB_IP]`

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

## Deploy, Scale, and Update Your Website on Google Kubernetes Engine

:star:

- Create a GKE cluster

  ```sh
  gcloud config set compute/zone us-central1-f
  gcloud services enable container.googleapis.com
  gcloud container clusters create fancy-cluster --num-nodes 3
  gcloud compute instances list
  ```

  - **Navigation menu** > **Kubernetes Engine** > **Clusters**

- Clone source repository

  ```sh
  cd ~
  git clone https://github.com/googlecodelabs/monolith-to-microservices.git
  cd ~/monolith-to-microservices
  ./setup.sh
  cd ~/monolith-to-microservices/monolith
  npm start
  ```

- Create Docker container with Cloud Build

  - single cmd to do build and submit

  ```sh
  gcloud services enable cloudbuild.googleapis.com
  cd ~/monolith-to-microservices/monolith
  gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 .
  ```

- Deploy container to GKE

  - Kubernetes represents applications as [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/), which are units that represent a container
  - The Pod is the smallest deployable unit in Kubernetes.
  - To deploy your application, create a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) resource. The Deployment manages multiple copies of your application, called replicas, and schedules them to run on the individual nodes in your cluster.
  - Deployments ensure this by creating a [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/). The ReplicaSet is responsible for making sure the number of replicas specified are always running.

  ```sh
  kubectl create deployment monolith --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0
  kubectl get all
  kubectl get pods
  kubectl get deploy
  kubectl get svc
  kubectl get rs
  
  #You can also combine the kubectl get pods,deployments
  ## debug
  kubectl describe pod monolith
  kubectl describe pod/monolith-787c4bbb5d-4zk66
  kubectl describe deployment monolith
  kubectl describe deployment.apps/monolith
  ```

  - **Navigation menu** > **Kubernetes Engine** > **Workloads**.
  - To see the full benefit of Kubernetes, simulate a server crash by deleting a pod and see what happens

  ```sh
  kubectl get pods
  kubectl delete pod/<POD_NAME> &
  kubectl get all
  ```

  - The ReplicaSet saw that the pod was terminating and triggered a new pod to keep up the desired replica count

- Expose GKE Deployment

  - You must explicitly expose your application to traffic from the Internet via a [Service](https://kubernetes.io/docs/concepts/services-networking/service/) resource. A Service provides networking and IP support to your application's Pods. GKE creates an external IP and a Load Balancer for your application.

  ```sh
  kubectl expose deployment monolith --type=LoadBalancer --port 80 --target-port 8080
  kubectl get service
  ```

- Scale GKE deployment up to 3 replicas

  ```sh
  kubectl scale deployment monolith --replicas=3
  kubectl get all
  ```

- Make changes to the website

  - **Scenario:** Your marketing team has asked you to change the homepage for your site. They think it should be more informative of who your company is and what you actually sell.

  ```sh
  ## update
  cd ~/monolith-to-microservices/react-app/src/pages/Home
  mv index.js.new index.js
  cd ~/monolith-to-microservices/react-app
  npm run build:monolith
  cd ~/monolith-to-microservices/monolith
  
  ## new cloud build
  gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0 .
  ```

- Update website with zero downtime

  - GKE's rolling update mechanism ensures that your application remains up and available even as the system replaces instances of your old container image with your new one across all the running replicas.

  ```sh
  ## update image
  kubectl set image deployment/monolith monolith=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0
  ## You will see 3 new pods being created and your old pods getting shut down
  kubectl get pods
  ```

- Cleanup

  ```sh
  ## delete repo
  cd ~rm -rf monolith-to-microservices
  
  ## delete container images
  gcloud container images delete gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:1.0.0 --quiet
  gcloud container images delete gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0 --quiet
  
  ## take all source archives from all builds and delete them from cloud storage
  gcloud builds list | awk 'NR > 1 {print $4}' | while read line; do gsutil rm $line; done
  
  ## delete gke service
  kubectl delete service monolith
  kubectl delete deployment monolith
  
  ## delete gke cluster
  gcloud container clusters delete fancy-cluster
  ```

## Migrating a Monolithic Website to Microservices on Google Kubernetes Engine

:star::star::star:

- Environment Setup > Clone Source Repository > Create a GKE Cluster

  ```sh
  gcloud config set compute/zone us-central1-f
  cd ~
  git clone https://github.com/googlecodelabs/monolith-to-microservices.git
  cd ~/monolith-to-microservices
  ./setup.sh
  gcloud services enable container.googleapis.com
  gcloud container clusters create fancy-cluster --num-nodes 3
  gcloud compute instances list
  ```

- Deploy Existing Monolith

  ```sh
  cd ~/monolith-to-microservices
  ./deploy-monolith.sh
  kubectl get service monolith
  ```

  - `http://[EXTERNAL-IP]`

- Migrate monolith to a microservice([ref](https://cloud.google.com/architecture/migrating-a-monolithic-app-to-microservices-gke#best_practices_for_microservices))

  - Now that you have a monolith website running on GKE, start breaking each service into a microservice.

  - Typically, a planning effort should take place to determine which services to break into smaller chunks, usually around specific parts of the application like business domain.

  - For this lab you will create an example and break out each service around the business domain: Orders, Products, and Frontend

  - Create new orders microservice

    ```sh
    ## cloud build
    cd ~/monolith-to-microservices/microservices/src/orders
    gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/orders:1.0.0 .
    
    # Navigation Menu > Cloud Build > History > Build ID
    ## deploy container to GKE
    kubectl create deployment orders --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/orders:1.0.0
    kubectl get all
    
    # Navigation menu > Kubernetes Engine > Workloads
    ## expose GKE container
    kubectl expose deployment orders --type=LoadBalancer --port 80 --target-port 8081
    kubectl get service orders
    ```

  - Reconfigure Monolith

    ```sh
    cd ~/monolith-to-microservices/react-app
    vim .env.monolith
    ```

    ```sh
    REACT_APP_ORDERS_URL=http://<ORDERS_IP_ADDRESS>/api/orders
    REACT_APP_PRODUCTS_URL=/service/products
    ```

    - test: `http://<ORDERS_IP_ADDRESS>/api/orders`

    ```sh
    ## rebuild
    npm run build:monolith
    
    ## cloud build
    cd ~/monolith-to-microservices/monolithgcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0 .
    
    ## redeploy container to GKE by update image
    kubectl set image deployment/monolith monolith=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:2.0.0
    ```

    - verify: `http://<MONOLITH_IP_ADDRESS>/orders`

  - Migrate Products to Microservice

    ```sh
    ## cloud build
    cd ~/monolith-to-microservices/microservices/src/products
    gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/products:1.0.0 .
    
    ## deploy container to GKE
    kubectl create deployment products --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/products:1.0.0
    ## expose GKE container
    kubectl expose deployment products --type=LoadBalancer --port 80 --target-port 8082
    kubectl get service products
    ```

    ```sh
    cd ~/monolith-to-microservices/react-app
    vim .env.monolith
    ```

    ```sh
    REACT_APP_ORDERS_URL=http://<ORDERS_IP_ADDRESS>/api/orders
    REACT_APP_PRODUCTS_URL=http://<PRODUCTS_IP_ADDRESS>/api/products
    ```

    - test: `http://<PRODUCTS_IP_ADDRESS>/api/products`

    ```sh
    ## rebuild
    npm run build:monolith
    
    ## cloud build
    cd ~/monolith-to-microservices/monolithgcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:3.0.0 .
    
    ## redeploy container to GKE by update image
    kubectl set image deployment/monolith monolith=gcr.io/${GOOGLE_CLOUD_PROJECT}/monolith:3.0.0
    ```

    - verify: `http://<MONOLITH_IP_ADDRESS>/products`

  - Migrate frontend to microservice

    ```sh
    ## update to microservices url
    cd ~/monolith-to-microservices/react-app
    cp .env.monolith .envnpm run build
    
    ## cloud build
    cd ~/monolith-to-microservices/microservices/src/frontend
    gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/frontend:1.0.0 .
    
    ## deploy container to GKE
    kubectl create deployment frontend --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/frontend:1.0.0
    
    ## expose GKE container
    kubectl expose deployment frontend --type=LoadBalancer --port 80 --target-port 8080
    kubectl get svc frontend
    ```

  - Delete the monolith

    ```sh
    kubectl delete deployment monolith
    kubectl delete service monolith
    ```

  - Test Your Work by: `http://<Frontend_IP_ADDRESS>`

## Build a Website on Google Cloud: Challenge Lab

- Challenge lab scenario: You have just started a new role at FancyStore, Inc. Your task is to take the company's existing monolithic e-commerce website and break it into a series of logically separated microservices. The existing monolith code is sitting in a GitHub repo, and you will be expected to containerize this app and then refactor it.

- **Task 1: Download the monolith code and build your container**

  ```sh
  cd ~
  git clone https://github.com/googlecodelabs/monolith-to-microservices.git
  cd ~/monolith-to-microservices
  ./setup.sh
  cd ~/monolith-to-microservices/monolith
  npm start
  gcloud services enable cloudbuild.googleapis.com
  gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/fancytest:1.0.0 .
  ```

- **Task 2: Create a kubernetes cluster and deploy the application**

  ```sh
  gcloud config set compute/zone us-central1-a
  gcloud services enable container.googleapis.com
  gcloud container clusters create fancy-cluster --num-nodes 3
  gcloud compute instances list
  kubectl create deployment fancytest --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/fancytest:1.0.0
  kubectl expose deployment fancytest --type=LoadBalancer --port 80 --target-port 8080
  kubectl get all
  ```

- **Task 3: Create a containerized version of your Microservices**

  ```sh
  cd ~/monolith-to-microservices/microservices/src/orders
  gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/orders:1.0.0 .
  cd ~/monolith-to-microservices/microservices/src/products
  gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/products:1.0.0 .
  ```

- **Task 4: Deploy the new microservices**

  ```sh
  kubectl create deployment orders --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/orders:1.0.0
  kubectl expose deployment orders --type=LoadBalancer --port 80 --target-port 8081
  kubectl create deployment products --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/products:1.0.0
  kubectl expose deployment products --type=LoadBalancer --port 80 --target-port 8082
  kubectl get all
  ```

- **Task 5: Configure the Frontend microservice**

  ```sh
  cd ~/monolith-to-microservices/react-app
  vim .env
  ```

  ```sh
  REACT_APP_ORDERS_URL=http://<ORDERS_IP_ADDRESS>/api/orders
  REACT_APP_PRODUCTS_URL=http://<PRODUCTS_IP_ADDRESS>/api/products
  ```

  ```sh
  npm run build
  ```

- **Task 6: Create a containerized version of the Frontend microservice**

  ```sh
  cd ~/monolith-to-microservices/microservices/src/frontend
  gcloud builds submit --tag gcr.io/${GOOGLE_CLOUD_PROJECT}/frontend:1.0.0 .
  ```

- **Task 7: Deploy the Frontend microservice**

  ```sh
  kubectl create deployment frontend --image=gcr.io/${GOOGLE_CLOUD_PROJECT}/frontend:1.0.0
  kubectl expose deployment frontend --type=LoadBalancer --port 80 --target-port 8080
  kubectl get all
  ```

  - `http://<Frontend_IP_ADDRESS>`
