# Google Cloud Solutions I: Scaling Your Infrastructure

[toc]

> The stable Helm chart repositories deprecated on 2020/11/13, use <https://artifacthub.io> to search new chart

## Autoscaling an Instance Group with Custom Cloud Monitoring Metrics

- In this lab you will create a [Compute Engine](https://cloud.google.com/compute/docs/) managed instance group that autoscales based on the value of a custom [Cloud Monitoring](https://cloud.google.com/monitoring/docs/) metric.

- The application includes:

  1. **Compute Engine instance template**: used to create `(5)` in `(4)`
  2. **Cloud Storage**: Store `(3)` and other script files
  3. **Compute Engine startup script**: It is installs and starts code automatically when an instance starts, and write values to `(6)`
  4. **Compute Engine instance group**: collection of `(5)`, autoscales based on `(6)`
  5. **Compute Engine instances**: Using `(3)` which is stored in `(2)` to startup.
  6. **Custom Cloud Monitoring metric**: A custom monitoring metric used to autoscale.

- ![App arch](https://cdn.qwiklabs.com/peFD70aHkYcyJdJHw7VfhmNAaG1jgPYJlIbcqMjcB9A%3D)

- Creatring the bucket(2)

  - **Navigation menu** > **Cloud Storage** > **Create bucket**

  - Name: PROJECT_ID

  - Create

  - Copy Compute Engine startup script(3) to bucket

    ```SH
    gsutil cp -r gs://spls/gsp087/* gs://${YOUR_BUCKET}
    ```

  - Refresh Bucket details page

    - `Startup.sh`: installs the necessary
    - `writeToCustomMetric.js`: Creates a custom monitoring metric whose value triggers scaling
    - `writeToCustomMetric.sh`: Continuously runs the `writeToCustomMetric.js`
    - `Config.json`:  Specifies the values for the custom monitoring metric

- Creating an instance template(1)

  - Create a template for the instances that are created in the instance group

  - **Navigation menu** > **Compute Engine** > **Instance templates** >  **Create Instance Template**

  - Name: autoscaling-instance01

  - Mangement > Metadata

    | Key                | Value                           |
    | :----------------- | :------------------------------ |
    | startup-script-url | `gs://[YOUR_BUCKET]/startup.sh` |
    | gcs-bucket         | `gs://[YOUR_BUCKET]`            |

  - Create

- Creating the instance group(4)

  - **Navigation menu** > **Compute Engine** > **Instance groups** > **Create instance group**
  - Name: autoscaling-instance-group-1
  - Instance template: autoscaling-instance01
  - Autoscaling mode: Don't autoscaling
  - Create

- Verifying that the instance group has been created

  - Wait to see the **green check** mark next to the new instance group you just created.

- Verifying that the Node.js script is running

  - **autoscaling-instance-group-1** > **instance name** > **Cloud Logging**
  - **Query preview** has `resource.type` & `resource.labels.instance_id`, now add `"nodeapp"` as line 3
  - Click **Run Query**
  - Check `Finished writing time series data` appear in the logs

- Configure autoscaling for the instance group based on the value of the custom metric.

  - **Compute Engine** > **Instance groups** > **autoscaling-instance-group-1** > **Configure**
  - Set **Autoscaling mode** to **Autoscale**
  - **Add new metric**
    - Metric Type: Stackdriver Monitoring metric
    - Metric export scope: Time series per instance
    - Metric identifier: custom.googleapis.com/appdemo_queue_depth_01
    - Utilization target: 150
    - Utilization target type: Gauge
      - The autoscaler should compute the average value of the data collected over the last few minutes and compare it to the target value.
    - Minimum number of instances: 1
    - Maximum number of instances: 3
    - Save

- Watching the instance group perform autoscaling

  - \> target, add comput engine instance; \< target, remove comput engine instance
  - **Compute Engine** > **Instance groups** > **builtin-igm** > **Monitoring** > Enable **Auto Refresh**

  > Since this group had a head start, you can see the autoscaling details about the instance group in the autoscaling graph. The autoscaler will take about five minutes to correctly recognize the custom metric and it can take up to ten minutes for the script to generate sufficient data to trigger the autoscaling behavior.

- [Autoscaling example](https://www.qwiklabs.com/focuses/611?parent=catalog#step12)

## Setting up Jenkins on Kubernetes Engine

- Prepare the Environment

  ```sh
  gcloud config set compute/zone us-east1-d
  git clone https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes.git
  cd continuous-deployment-on-kubernetes
  
  ## create k8s cluster
  gcloud container clusters create jenkins-cd \
  --num-nodes 2 \
  --machine-type n1-standard-2 \
  --scopes "https://www.googleapis.com/auth/projecthosting,cloud-platform"
  
  gcloud container clusters list
  gcloud container clusters get-credentials jenkins-cd
  kubectl cluster-info
  ```

- Configure Helm

  - Helm is a package manager that makes it easy to configure and deploy Kubernetes applications.

  ```sh
  helm version
  helm repo add stable https://charts.helm.sh/stable
  helm repo update
  ```

- Configure and Install Jenkins

  - You will use the `jenkins/values.yaml`  to add the Google Cloud specific plugin necessary to use service account credentials to reach your Cloud Source Repository.
  - Jenkins plugin to run dynamic agents in a Kubernetes cluster, the plugin creates a Kubernetes Pod for each agent started, and stops it after each build.

  ```sh
  ## name=cd, chart=stable/jenkins
  helm install cd stable/jenkins -f jenkins/values.yaml --version 1.2.2 --wait
  kubectl get pods
  
  ## port forwarding
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
  kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
  
  ## 
  kubectl get svc
  ```

- Connect to Jenkins

  - **Web Preview** > **Preview on port 8080**
    - username: admin
    - password: password get by: `printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo`
  - You can try to set a pipeline and build it, and use `kubectl get pods` to see the agent start.

## Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine

- This hands-on lab shows you how to create a continuous delivery pipeline using Google Kubernetes Engine, Google Cloud Source Repositories, Google Cloud Container Builder, and Spinnaker.

- Pipeline architecture

  - To continuously deliver application updates to your users, you need an automated process that reliably builds, tests, and updates your software.
  - Code changes should automatically flow through a pipeline that includes artifact creation, unit testing, functional testing, and production rollout.
  - In some cases, you want a code update to apply to only a subset of your users, so that it is exercised realistically before you push it to your entire user base.
  - If one of these [canary](https://martinfowler.com/bliki/CanaryRelease.html) releases proves unsatisfactory, your automated procedure must be able to quickly roll back the software changes.

- Application delivery pipeline flow

  1. Developer
     1. Change code
     2. Create a Git tag and push to repo
  2. Container Builder
     1. Detect new tag
     2. Build docker image
     3. Run unit tests
     4. Push Docker image
  3. Spinnaker
     1. Detect new image
     2. Deploy to canary
     3. Functional tests of canary deployment
     4. Manual approval
     5. Deploy to production

- Set up your environment

  ```sh
  gcloud config set compute/zone us-central1-f
  ## cluster need 5~10 mins to be created
  gcloud container clusters create spinnaker-tutorial \
      --machine-type=n1-standard-2
      
  ## create a Spinnaker service account
  gcloud iam service-accounts create spinnaker-account \
      --display-name spinnaker-account
  export SA_EMAIL=$(gcloud iam service-accounts list \
      --filter="displayName:spinnaker-account" \
      --format='value(email)')
  export PROJECT=$(gcloud info --format='value(config.project)')
  
  ## allowing Spinnaker to store data in Cloud Storage. 
  gcloud projects add-iam-policy-binding $PROJECT \
      --role roles/storage.admin \
      --member serviceAccount:$SA_EMAIL
      
  ## create key    
  gcloud iam service-accounts keys create spinnaker-sa.json \
       --iam-account $SA_EMAIL
  ```

- Set up Cloud Pub/Sub to trigger Spinnaker pipelines

  ```sh
  ## create the Cloud Pub/Sub topic for notifications from Container Registry.
  gcloud pubsub topics create projects/$PROJECT/topics/gcr
  
  ## create a subscription that Spinnaker can read from to receive notifications of images being pushed.
  gcloud pubsub subscriptions create gcr-triggers \
      --topic projects/${PROJECT}/topics/gcr
  
  ## give Spinnaker permissions to read subscription
  gcloud beta pubsub subscriptions add-iam-policy-binding gcr-triggers \
      --role roles/pubsub.subscriber --member serviceAccount:$SA_EMAIL
  ```

- Deploying Spinnaker using Helm

  ```sh
  ## grant Helm the cluster-admin role
  kubectl create clusterrolebinding user-admin-binding \
      --clusterrole=cluster-admin --user=$(gcloud config get-value account)
  ## grant Spinnaker the cluster-admin role
  kubectl create clusterrolebinding --clusterrole=cluster-admin \
      --serviceaccount=default:default spinnaker-admin
  
  ## add stable chrart to Helm's repo
  helm repo add stable https://charts.helm.sh/stable
  helm repo update
  
  ## create a bucket for Spinnaker to store its pipeline configuration
  export BUCKET=$PROJECT-spinnaker-config
  gsutil mb -c regional -l us-central1 gs://$BUCKET   
  ```

  ```SH
  ## create spinnaker-config.yaml by run following code
  export SA_JSON=$(cat spinnaker-sa.json)
  cat > spinnaker-config.yaml <<EOF
  gcs:
    enabled: true
    bucket: $BUCKET
    project: $PROJECT
    jsonKey: '$SA_JSON'
  
  dockerRegistries:
  - name: gcr
    address: https://gcr.io
    username: _json_key
    password: '$SA_JSON'
    email: 1234@5678.com
  
  # Disable minio as the default storage backend
  minio:
    enabled: false
  
  # Configure Spinnaker to enable GCP services
  halyard:
    spinnakerVersion: 1.19.4
    image:
      repository: us-docker.pkg.dev/spinnaker-community/docker/halyard
      tag: 1.32.0
      pullSecrets: []
    additionalScripts:
      create: true
      data:
        enable_gcs_artifacts.sh: |-
          \$HAL_COMMAND config artifact gcs account add gcs-$PROJECT --json-path /opt/gcs/key.json
          \$HAL_COMMAND config artifact gcs enable
        enable_pubsub_triggers.sh: |-
          \$HAL_COMMAND config pubsub google enable
          \$HAL_COMMAND config pubsub google subscription add gcr-triggers \
            --subscription-name gcr-triggers \
            --json-path /opt/gcs/key.json \
            --project $PROJECT \
            --message-format GCR
  EOF
  ```

  ```SH
  ## deploy the Spinnaker chart
  helm install -n default cd stable/spinnaker -f spinnaker-config.yaml \
             --version 2.0.0-rc9 --timeout 10m0s --wait
  export DECK_POD=$(kubectl get pods --namespace default -l "cluster=spin-deck" -o jsonpath="{.items[0].metadata.name}")
  
  ## port forwarding
  kubectl port-forward --namespace default $DECK_POD 8080:9000 >> /dev/null &
  ```

  - **Web Preview** > **Preview on port 8080**

- Building the Docker image

  ```sh
  ## download the sample
  gsutil -m cp -r gs://spls/gsp114/sample-app.tar .
  mkdir sample-app
  tar xvf sample-app.tar -C ./sample-app
  cd sample-app
  
  ## git commit
  git config --global user.email "$(gcloud config get-value core/account)"
  git config --global user.name student
  git init
  git add .
  git commit -m "Initial commit"
  
  ## create gcp remote repo and push
  gcloud source repos create sample-app
  git config credential.helper gcloud.sh
  git remote add origin https://source.developers.google.com/p/$PROJECT/r/sample-app
  git push origin master
  ```

  - **Navigation Menu** > **Source Repositories** > **sample.app**

- Configure your cloud build triggers

  - **Navigation menu** > **Cloud Build** > **Triggers** > **Create trigger**
  - **Name**: sample-app-tags
  - **Event**: Push new tag
  - **Repository**: sample-app
  - **Tag**: v1.*
  - **Configuration**: Cloud Build configuration file (yaml or json)
  - **Cloud Build configuration file location**: `/cloudbuild.yaml`
  - CREATE

- Prepare your Kubernetes Manifests for use in Spinnaker

  ```SH
  ## Creates a Cloud Storage bucket that will be populated with your manifests during the CI process in Cloud Build
  gsutil mb -l us-central1 gs://$PROJECT-kubernetes-manifests
  sed -i s/PROJECT/$PROJECT/g k8s/deployments/*
  git commit -a -m "Set project ID"
  ```

- Build image

  ```sh
  git tag v1.0.0git push --tags
  ```

  - **Navigation menu** > **Cloud Build** > **History**
  - Stay on this page and wait  green check

- Configuring your deployment pipelines

  ```sh
  ## download spin CLI for managing Spinnaker
  curl -LO https://storage.googleapis.com/spinnaker-artifacts/spin/1.14.0/linux/amd64/spin
  chmod +x spin
  
  ## create the application
  ./spin application save --application-name sample \
                          --owner-email "$(gcloud config get-value core/account)" \
                          --cloud-providers kubernetes \
                          --gate-endpoint http://localhost:8080/gate
                          
  ## upload an example pipleline
  sed s/PROJECT/$PROJECT/g spinnaker/pipeline-deploy.json > pipeline.json
  ./spin pipeline save --gate-endpoint http://localhost:8080/gate -f pipeline.json
  ```

- Manually Trigger and View your pipeline execution

  - **Spinnaker UI** > **Applications** > **sample** > **Pipelines** > **Start Manual Execution**> **Run**
  - **Execution Details** > Click a stage to see details about it
  - After **3 to 5 minutes** the integration test phase completes and the pipeline requires manual approval to continue the deployment.
  - Hover over the yellow "person" icon and click **Continue**.
  - **Infrastructure** > **Load Balancers** > **service sample-frontend-production Default**
  - copy **Ingress** IP and paste it to new tab
  - Refresh many times, you will see production version and canary version(4:1)

- Triggering your pipeline from code changes

  ```SH
  ## change color from orange to blue
  sed -i 's/orange/blue/g' cmd/gke-info/common-service.go
  git commit -a -m "Change color to blue"
  git tag v1.0.1
  git push --tags
  ```

  - **Navigation menu** > **Cloud Build** > **History**
  - Stay on this page and wait  green check
  - **Spinnaker UI** > **Applications** > **sample** > **Pipelines**
  - When the deployment is paused, waiting to roll out to production, return to the web app page and start refreshing the tab.
  - You should see the new, blue version(**Canary**) of your app appear about every fifth time you refresh.
  - **Continue** the deployment.
  - When the pipeline completes, all versions will show blue background

- Rollback

  ```sh
  git revert v1.0.1
  git tag v1.0.2
  git push --tags
  ```

  - When the build and then the pipeline completes, all versions will show orange background

## Deploying a Fault-Tolerant Microsoft Active Directory Environment

- This lab is part of a series aimed at helping you deploy a highly available Windows architecture on Google Cloud with Microsoft Active Directory (AD), SQL Server, and Internet Information Services (IIS). In this lab you set up a redundant pair of Windows Domain Controllers (DC) with AD using a new Virtual Private Cloud (VPC) network and multiple subnets.

- Initializing common variables

  ```sh
  export region1=us-central1
  export region2=us-west2
  export zone_1=${region1}-b
  export zone_2=${region2}-c
  export vpc_name=webappnet
  export project_id=$(gcloud config get-value project)
  gcloud config set compute/region ${region1}
  ```

- Creating the network infrastructure

  ```sh
  ## VPC network
  gcloud compute networks create ${vpc_name}  \
      --description "VPC network to deploy Active Directory" \
      --subnet-mode custom
  
  ## subnet
  gcloud compute networks subnets create private-ad-zone-1 \
      --network ${vpc_name} \
      --range 10.1.0.0/24 \
      --region ${region1}
      
  gcloud compute networks subnets create private-ad-zone-2 \
      --network ${vpc_name} \
      --range 10.2.0.0/24 \
      --region ${region2}
      
  ## internal firewall rule between two subnet
  gcloud compute firewall-rules create allow-internal-ports-private-ad \
      --network ${vpc_name} \
      --allow tcp:1-65535,udp:1-65535,icmp \
      --source-ranges  10.1.0.0/24,10.2.0.0/24
  
  ## RDP firewall rule
  gcloud compute firewall-rules create allow-rdp \
      --network ${vpc_name} \
      --allow tcp:3389 \
      --source-ranges 0.0.0.0/0
  ```

- Creating the first domain controller

  ```sh
  gcloud compute instances create ad-dc1 --machine-type n1-standard-2 \
      --boot-disk-type pd-ssd \
      --boot-disk-size 50GB \
      --image-family windows-2016 --image-project windows-cloud \
      --network ${vpc_name} \
      --zone ${zone_1} --subnet private-ad-zone-1 \
      --private-network-ip=10.1.0.100
      
  ## create password
  gcloud compute reset-windows-password ad-dc1 --zone ${zone_1} --quiet --user=admin
  ## Save the ip-address, username and password returned in Cloud Shell 
  ```

- Creating the second domain controller

  ```sh
  export region2=us-west2
  export zone_2=${region2}-c
  export vpc_name=webappnet
  export project_id=$(gcloud config get-value project)
  gcloud config set compute/region ${region2}
  gcloud compute instances create ad-dc2 --machine-type n1-standard-2 \
      --boot-disk-size 50GB \
      --boot-disk-type pd-ssd \
      --image-family windows-2016 --image-project windows-cloud \
      --can-ip-forward \
      --network ${vpc_name} \
      --zone ${zone_2} \
      --subnet private-ad-zone-2 \
      --private-network-ip=10.2.0.100
  ```

  ==Shelve this class, cause i do not have RDP software==

## Deploying Memcached on Kubernetes Engine

- In this lab you'll learn how to deploy a cluster of distributed [Memcached](https://memcached.org/) servers on [Kubernetes Engine](https://cloud.google.com/kubernetes-engine/) using [Kubernetes](https://kubernetes.io/), [Helm](https://helm.sh/), and [Mcrouter](https://github.com/facebook/mcrouter).

- Memcached is one of the most popular open source, multi-purpose caching systems. It usually serves as a temporary store for frequently used data to speed up web applications and lighten database loads.

  - **Simplicity**: like a large hash table
  - **Speed**: holds cache data  in RAM

- Memcached is a distributed system that allows its hash table capacity to scale horizontally across a pool of servers. The routing and load balancing between the servers must be done at the client level.

  - The same server is always selected for the same key.
  - Memory usage is evenly balanced between the servers.
  - A minimum number of keys are relocated when the pool of servers is reduced or expanded.

- Deploying a Memcached service

  ```sh
  ## 3 nodes gke cluster
  gcloud container clusters create demo-cluster --num-nodes 3 --zone us-central1-f
  
  ## configure helm
  helm repo add stable https://charts.helm.sh/stable
  helm repo update
  
  ## deploy mycache memcached helm chart
  helm install mycache stable/memcached --set replicaCount=3
  kubectl get pods
  ```

- Discovering Memcached service endpoints

  - Memcached Helm chart uses a headless service. A headless service exposes IP addresses for all of its pods so that they can be individually discovered
  - DNS record: mycache-memcached.default.svc.cluster.local(`[SERVICE_NAME].[NAMESPACE].svc.cluster.local`)

  ```sh
  ## Retrieve the endpoints' IP addresses:
  kubectl get endpoints mycache-memcached
  
  ## Anothe way, use nslookup
  kubectl run -it --rm alpine --image=alpine:3.6 --restart=Never nslookup mycache-memcached.default.svc.cluster.local
  # [POD_NAME].[SERVICE_NAME].[NAMESPACE].svc.cluster.local
  
  ## Another way, use python
  >>> import socket
  >>> print(socket.gethostbyname_ex('mycache-memcached.default.svc.cluster.local'))
  >>> exit()
  ```

  - Test the deployment by opening a telnet session

  ```sh
  ## telnet
  kubectl run -it --rm alpine --image=alpine:3.6 --restart=Never telnet mycache-memcached-0.mycache-memcached.default.svc.cluster.local 11211
  
  ## test
  
  # set key compress expiration length
  set mykey 0 0 5
  # value
  hello
  # get key
  get mykey
  
  quit
  ```

- Implementing the service discovery logic

  - ![!mg](https://cdn.qwiklabs.com/sc0rg%2BcEvqxI8%2BojmPOzhOnlLCCrsw%2F8YxhMAZsmlgw%3D)
    1. The application queries `kube-dns` for the DNS record of `mycache-memcached.default.svc.cluster.local`.
    2. The application retrieves the IP addresses associated with that record.
    3. The application instantiates a new Memcached client and provides it with the retrieved IP addresses.
    4. The Memcached client's integrated load balancer connects to the Memcached servers at the given IP addresses.

  ```sh
  ## using pythonkubectl run -it --rm python --image=python:3.6-alpine --restart=Never shpip install pymemcachepython
  ```

  ```python
  import socket
  from pymemcache.client.hash import HashClient
  
  ## 1
  _, _, ips = socket.gethostbyname_ex('mycache-memcached.default.svc.cluster.local')
  ## 2
  servers = [(ip, 11211) for ip in ips]
  ## 3
  client = HashClient(servers, use_pooling=True)
  ## 4
  client.set('mykey', 'hello')
  client.get('mykey')
  ```

- Enabling connection pooling using Mcrouter

  - In particular, As your caching needs grow, the large number of open connections from Memcached clients might place a heavy load on the servers.
  - To reduce the number of open connections, you must introduce a proxy to enable connection pooling
  - ![img](https://cdn.qwiklabs.com/McgEiSHBkoDtEpmtZ30aUFD2s%2Fwf1v9vsdIAo%2FRWyY8%3D)
  - [Mcrouter](https://github.com/facebook/mcrouter) (pronounced "mick router"), a powerful open source Memcached proxy, enables connection pooling.

  ```sh
  ## delete mycache helm chart
  helm delete mycache
  ## deploy new memcached and mcrouter using mcrouter helm chart
  helm install mycache stable/mcrouter --set memcached.replicaCount=3
  ```

  ```sh
  ## test connect: telnet
  MCROUTER_POD_IP=$(kubectl get pods -l app=mycache-mcrouter -o jsonpath="{.items[0].status.podIP}")
  kubectl run -it --rm alpine --image=alpine:3.6 --restart=Never telnet $MCROUTER_POD_IP 5000
  
  ## cmd
  set anotherkey 0 0 15
  Mcrouter is fun
  get anotherkey
  quit
  ```

- Reducing latency

  - To increase resilience, it is common practice to use a cluster with multiple nodes. However, using multiple nodes also brings the risk of increased latency caused by heavier network traffic between nodes.

  - You can reduce the latency risk by connecting client application pods only to a Memcached proxy pod that is on the same node.

  - ![Colocating proxy pods](https://cdn.qwiklabs.com/lrOApI5zGzaauVVZg07ahgD0os8kXUYS7dPaDO6TzTI%3D)

  - In a production environment, you would create this configuration as follows:

    1. Ensure that each node contains one running proxy pod.A common approach is to deploy the proxy pods with a [DaemonSet controller](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset).(Mcrouter Helm chart did it)

    2. Set a [hostPort](https://v1-7.docs.kubernetes.io/docs/api-reference/v1.7/#containerport-v1-core) value in the proxy container's Kubernetes parameters to make the node listen to that port and redirect traffic to the proxy.(Mcrouter Helm chart did it for `port 5000`)

    3. Expose the node name as an environment variable inside the application pods by using the `spec.env` entry and selecting the `spec.nodeName` `fieldRef` value.

       ```sh
       cat <<EOF | kubectl create -f -
       apiVersion: apps/v1
       kind: Deployment
       metadata:
         name: sample-application-py
       spec:
         replicas: 5
         selector:
           matchLabels:
             app: sample-application-py
         template:
           metadata:
             labels:
               app: sample-application-py
           spec:
             containers:
               - name: python
                 image: python:3.6-alpine
                 command: [ "sh", "-c"]
                 args:
                 - while true; do sleep 10; done;
                 env:
                   - name: NODE_NAME
                     valueFrom:
                       fieldRef:
                         fieldPath: spec.nodeName
       EOF
       ```

  - Connecting the pods

    ```sh
    POD=$(kubectl get pods -l app=sample-application-py -o jsonpath="{.items[0].metadata.name}")
    NODE_NAME=$(kubectl exec -it $POD -- sh -c 'echo $NODE_NAME' | tr -d '\r')
    
    ## use telnet
    kubectl run -it --rm alpine --image=alpine:3.6 --restart=Never telnet $NODE_NAME 5000
    get anotherkey
    quit
    
    ## use python
    kubectl exec -it $POD -- sh
    pip install pymemcache
    python
    ```

    ```python
    import os
    from pymemcache.client.base import Client
    
    NODE_NAME = os.environ['NODE_NAME']
    client = Client((NODE_NAME, 5000))
    client.set('some_key', 'some_value')
    result = client.get('some_key')
    result
    result = client.get('anotherkey')
    result
    ```
