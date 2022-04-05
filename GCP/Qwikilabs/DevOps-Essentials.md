# DevOps Essentials

## Cloud Source Repositories: Qwik Start

```sh
# Create a new repository
gcloud source repos create REPO_DEMO

# Clone the new repository into your Cloud Shell session
gcloud source repos clone REPO_DEMO

# Push to the Cloud Source Repository
cd REPO_DEMO
echo 'Hello World!' > myfile.txt
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
git add myfile.txt
git commit -m "First file using Cloud Source Repositories"
git push origin master
```

-  **Navigation menu** > **Source Repositories**

## Managing Deployments Using Kubernetes Engine

The following exercises practice some common use cases for heterogeneous deployments, along with well-architected approaches using Kubernetes and other infrastructure resources to accomplish them.

```sh
# Get the sample code for creating and running containers and deployments
gsutil -m cp -r gs://spls/gsp053/orchestrate-with-kubernetes .
cd orchestrate-with-kubernetes/kubernetes

# Create a cluster with five n1-standard-1 nodes
gcloud config set compute/zone us-central1-a
gcloud container clusters create bootcamp --num-nodes 5 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
```

### Create a deployment

```sh
kubectl explain deployment
kubectl explain deployment --recursive
kubectl explain deployment.metadata.name
```

```sh
vi deployments/auth.yaml
```

```yaml
...
containers:
- name: auth
  image: "kelseyhightower/auth:1.0.0"
...
```

```sh
cat deployments/auth.yaml

# auth
kubectl create -f deployments/auth.yaml
kubectl create -f services/auth.yaml

# a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features.
kubectl get deployments
kubectl get replicasets
kubectl get pods

# hello
kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml

# frontend
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml
```

### Rolling update and Rollback an update

When a Deployment is updated with a new version, it creates a new ReplicaSet and slowly increases the number of replicas in the new ReplicaSet as it decreases the replicas in the old ReplicaSet.

```sh
# rolling update
kubectl edit deployment hello
```

```yaml
...
containers:
  image: kelseyhightower/hello:2.0.0
...
```

```sh
# Get updating status and history
kubectl get replicaset
kubectl rollout history deployment/hello

# Pause a rolling update and get status
kubectl rollout pause deployment/hello
kubectl rollout status deployment/hello
kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'

# Resume a rolling update and get status
kubectl rollout resume deployment/hello
kubectl rollout status deployment/hello
```

```sh
# rollback
kubectl rollout undo deployment/hello
kubectl rollout history deployment/hello
kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
kubectl rollout status deployment/hello
```

### Canary deployments

When you want to test a new deployment in production with a subset of your users, use a canary deployment. 

A canary deployment consists of a separate deployment with your new version and a service that targets both your normal, stable deployment as well as your canary deployment.

![48190cf58fdf2eeb.png](https://cdn.qwiklabs.com/qSrgIP5FyWKEbwOk3PMPAALJtQoJoEpgJMVwauZaZow%3D)

```sh
cat deployments/hello-canary.yaml
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
        track: canary
        # Use ver 2.0.0 so it matches version on service selector
        version: 2.0.0
    spec:
      containers:
        - name: hello
          image: kelseyhightower/hello:2.0.0
          ports:
            - name: http
              containerPort: 80
            - name: health
              containerPort: 81
```

```sh
kubectl create -f deployments/hello-canary.yaml
kubectl get deployments

# On the hello service, the selector uses the app:hello selector which will match pods in both the prod deployment and canary deployment. However, because the canary deployment has a fewer number of pods, it will be visible to fewer users.(1/4)
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
```

### Blue-green deployments

Rolling updates are ideal because they allow you to deploy an application slowly with minimal overhead, minimal performance impact, and minimal downtime. There are instances where it is beneficial to modify the load balancers to point to that new version only after it has been fully deployed. In this case, blue-green deployments are the way to go.

Kubernetes achieves this by creating two separate deployments; one for the old "blue" version and one for the new "green" version. Use your existing `hello` deployment for the "blue" version. ==The deployments will be accessed via a Service which will act as the router. Once the new "green" version is up and running, you'll switch over to using that version by updating the Service.==A major downside of blue-green deployments is that you will need to have at least 2x the resources in your cluster necessary to host your application.

![9e624196fdaf4534.png](https://cdn.qwiklabs.com/POW8Q247ZKNY%2ByHIartCsoEu8MAih7k4u1twusCx6pw%3D)



```sh
kubectl apply -f services/hello-blue.yaml
kubectl create -f deployments/hello-green.yaml
```

```sh
# Once you have a green deployment and it has started up properly, verify that the current version of 1.0.0 is still being used:
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
> {"version":"1.0.0"}

# Now, update the service to point to the new version:
kubectl apply -f services/hello-green.yaml
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
> {"version":"2.0.0"}
```

## Deploy Kubernetes Load Balancer Service with Terraform

In Terraform, a Provider is the logical abstraction of an upstream API. This lab will show you how to setup a Kubernetes cluster and deploy Load Balancer type NGINX service on it.

### Kubernetes Services

A service is a grouping of pods that are running on the cluster.Each service has a pod label query which defines the pods which will process data for the service

### Why Terraform?

- You can use the same [configuration language](https://www.terraform.io/docs/configuration/syntax.html) to provision the Kubernetes infrastructure and to deploy applications into it.
- **Drift detection**:`terraform plan`
- **Full lifecycle management**: offers a single command to create, update, and delete tracked resources
- **Synchronous feedback** &**[Graph of relationships](https://www.terraform.io/docs/internals/graph.html)**

### Code

```sh
gsutil -m cp -r gs://spls/gsp233/* .
cd tf-gke-k8s-service-lb
cat main.tf
cat k8s.tf
```

```hcl
variable "region" {
  default = "us-west1"
}

variable "location" {
  default = "us-west1-b"
}

variable "network_name" {
  default = "tf-gke-k8s"
}

provider "google" {
  region = var.region
}

resource "google_compute_network" "default" {
  name                    = var.network_name
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "default" {
  name                     = var.network_name
  ip_cidr_range            = "10.127.0.0/20"
  network                  = google_compute_network.default.self_link
  region                   = var.region
  private_ip_google_access = true
}

data "google_client_config" "current" {
}

data "google_container_engine_versions" "default" {
  location = var.location
}

resource "google_container_cluster" "default" {
  name               = var.network_name
  location           = var.location
  initial_node_count = 3
  min_master_version = data.google_container_engine_versions.default.latest_master_version
  network            = google_compute_subnetwork.default.name
  subnetwork         = google_compute_subnetwork.default.name

  // Use legacy ABAC until these issues are resolved: 
  //   https://github.com/mcuadros/terraform-provider-helm/issues/56
  //   https://github.com/terraform-providers/terraform-provider-kubernetes/pull/73
  enable_legacy_abac = true

  // Wait for the GCE LB controller to cleanup the resources.
  // Wait for the GCE LB controller to cleanup the resources.
  provisioner "local-exec" {
    when    = destroy
    command = "sleep 90"
  }
}

output "network" {
  value = google_compute_subnetwork.default.network
}

output "subnetwork_name" {
  value = google_compute_subnetwork.default.name
}

output "cluster_name" {
  value = google_container_cluster.default.name
}

output "cluster_region" {
  value = var.region
}

output "cluster_location" {
  value = google_container_cluster.default.location
}
```

```hcl
provider "kubernetes" {
  version = "~> 1.10.0"
  host    = google_container_cluster.default.endpoint
  token   = data.google_client_config.current.access_token
  client_certificate = base64decode(
    google_container_cluster.default.master_auth[0].client_certificate,
  )
  client_key = base64decode(google_container_cluster.default.master_auth[0].client_key)
  cluster_ca_certificate = base64decode(
    google_container_cluster.default.master_auth[0].cluster_ca_certificate,
  )
}

resource "kubernetes_namespace" "staging" {
  metadata {
    name = "staging"
  }
}

resource "google_compute_address" "default" {
  name   = var.network_name
  region = var.region
}

resource "kubernetes_service" "nginx" {
  metadata {
    namespace = kubernetes_namespace.staging.metadata[0].name
    name      = "nginx"
  }

  spec {
    selector = {
      run = "nginx"
    }

    session_affinity = "ClientIP"

    port {
      protocol    = "TCP"
      port        = 80
      target_port = 80
    }

    type             = "LoadBalancer"
    load_balancer_ip = google_compute_address.default.address
  }
}

resource "kubernetes_replication_controller" "nginx" {
  metadata {
    name      = "nginx"
    namespace = kubernetes_namespace.staging.metadata[0].name

    labels = {
      run = "nginx"
    }
  }

  spec {
    selector = {
      run = "nginx"
    }

    template {
      metadata {
          name = "nginx"
          labels = {
              run = "nginx"
          }
      }

      spec {
        container {
            image = "nginx:latest"
            name  = "nginx"

            resources {
                limits {
                    cpu    = "0.5"
                    memory = "512Mi"
                }

                requests {
                    cpu    = "250m"
                    memory = "50Mi"
                }
            }
        }       
      }
    }
  }
}

output "load-balancer-ip" {
  value = google_compute_address.default.address
}
```

### Initialize and install dependencies

```sh
terraform init
terraform apply
yes

# verify
gcloud config set compute/zone us-west1-b
gcloud container clusters get-credentials tf-gke-k8s
kubectl get all
```

- **Navigation menu** > **Kubernetes Engine** >  `tf-gke-k8s` > **Services & Ingress** > `nginx`
- Click the **Endpoints** IP address

## Troubleshooting Workloads on GKE for Site Reliability Engineers

[oldnote]()

## Continuous Delivery with Jenkins in Kubernetes Engine

In this lab, you will learn how to set up a continuous delivery pipeline with `Jenkins` on Kubernetes engine. Jenkins is the go-to automation server used by developers who frequently integrate their code in a shared repository. The solution you'll build in this lab will be similar to the following diagram:![overview.png](https://cdn.qwiklabs.com/1b%2B9D20QnfRjAF8c6xlXmexot7TDcOsYzsRwp%2FH4ErE%3D)

ref: <https://cloud.google.com/architecture/jenkins-on-kubernetes-engine>

###  Setup Jenkins

```sh
# source code
gcloud config set compute/zone us-east1-d
gsutil cp gs://spls/gsp051/continuous-deployment-on-kubernetes.zip .
unzip continuous-deployment-on-kubernetes.zip
cd continuous-deployment-on-kubernetes

# k8s cluster
gcloud container clusters create jenkins-cd \
--num-nodes 2 \
--machine-type n1-standard-2 \
--scopes "https://www.googleapis.com/auth/source.read_write,cloud-platform"
gcloud container clusters list
gcloud container clusters get-credentials jenkins-cd
kubectl cluster-info

# helm
helm repo add jenkins https://charts.jenkins.io
helm repo update
## When installing Jenkins, a values file can be used as a template to provide values that are necessary for setup.
gsutil cp gs://spls/gsp330/values.yaml jenkins/values.yaml
helm install cd jenkins/jenkins -f jenkins/values.yaml --wait

kubectl get pods
# Configure Jenkins service account
kubectl create clusterrolebinding jenkins-deploy --clusterrole=cluster-admin --serviceaccount=default:cd-jenkins


export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
kubectl get svc
# You are using the Kubernetes Plugin so that our builder nodes will be automatically launched as necessary when the Jenkins master requests them.
```

### Connect to Jenkins

- **Web Preview** > **Preview on port 8080**
- username: admin
- password:

```sh
printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
```

### Understanding the Application

You'll deploy the sample application, `gceme`, in your continuous deployment pipeline. The application is written in the Go language and is located in the repo's sample-app directory. When you run the gceme binary on a Compute Engine instance, the app displays the instance's metadata in an info card.

![90fb779a65809b9e.png](https://cdn.qwiklabs.com/ceqFX6Vwtd12NSBtnNhrkemKfRSLHbCmFZiLn8WmC98%3D)

The application mimics a microservice by supporting two operation modes.

- In **backend mode**: gceme listens on port 8080 and returns Compute Engine instance metadata in JSON format.
- In **frontend mode**: gceme queries the backend gceme service and renders the resulting JSON in the user interface.

![fc22b68ab20dfe0e.png](https://cdn.qwiklabs.com/P1T5JBWWprA4iLf%2FB5%2BO6as7otLE25YBde57gzZwSz4%3D)

### Deploying the Application

You will deploy the application into two different environments:

- **Production**: The live site that your users access.
- **Canary**: A smaller-capacity site that receives only a percentage of your user traffic. Use this environment to validate your software with live traffic before it's released to all of your users.

```sh
cd sample-app

# namespce
kubectl create ns production

kubectl apply -f k8s/production -n production
kubectl apply -f k8s/canary -n production
kubectl apply -f k8s/services -n production
```

By default, only one replica of the frontend is deployed. Use the `kubectl scale`command to ensure that there are at least 4 replicas running at all times.

```sh
# Scale Frontend prodction to 4 replicas
kubectl scale deployment gceme-frontend-production -n production --replicas 4
# Frontend 4:1
kubectl get pods -n production -l app=gceme -l role=frontend
# Backend 1:1
kubectl get pods -n production -l app=gceme -l role=backend

kubectl get service gceme-frontend -n production
export FRONTEND_SERVICE_IP=$(kubectl get -o jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
curl http://$FRONTEND_SERVICE_IP/version
> 1.0.0
```

### Creating the Jenkins Pipeline

```sh
gcloud source repos create default
git init
git config credential.helper gcloud.sh
git remote add origin https://source.developers.google.com/p/$DEVSHELL_PROJECT_ID/r/default
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
git add .
git commit -m "Initial commit"
git push origin master
```

#### **Adding your service account credentials**

- Jenkins user interface > **Manage Jenkins** > **Manage Credentials** >Store:  **Jenkins** > **Global credentials (unrestricted)** > **Add Credentials**
  - Kind : **Google Service Account from metadata**
  - **OK**

#### **Creating the Jenkins job**

- Jenkins user interface > **New Item**
  - **sample-app**
  - **Multibranch Pipeline**
  - **Branch Sources** > **Add Source** > **git**
    - Project Repository: <https://source.developers.google.com/p/[PROJECT_ID]/r/default>
    - Credentials: **qwiklabs-XXX service account**
  - **Scan Multibranch Pipeline Triggers**
    - Periodically if not otherwise run
    - Interval 1 minute
  - **Save**

### Creating the Development Environment

```sh
git checkout -b new-feature
vi Jenkinsfile
# Add your PROJECT_ID to the REPLACE_WITH_YOUR_PROJECT_ID value.

vi html.go
vi main.go
```

```html
<!--<div class="card blue">-->
<div class="card orange">
```

```go
//const version string = "1.0.0"
const version string = "2.0.0"
```

#### Kick off Deployment

```
git add Jenkinsfile html.go main.go
git commit -m "Version 2.0.0"
git push origin new-feature
```

- Jenkins user interface > **part of sample-app » new-feature #1** > Console Output

```sh
# Once that's all taken care of, start the proxy in the background:
kubectl proxy &
curl \
http://localhost:8001/api/v1/namespaces/new-feature/services/gceme-frontend:80/proxy/version
> 2.0.0
```

> **Note:** In a development scenario, you wouldn't use a public-facing load balancer. To help secure your application, you can use [kubectl proxy](https://kubernetes.io/docs/tasks/extend-kubernetes/http-proxy-access-api/). The proxy authenticates itself with the Kubernetes API and proxies requests from your local machine to the service in the cluster without exposing your service to the Internet.

### Deploying a Canary Release

You have verified that your app is running the latest code in the development environment, so now deploy that code to the canary environment.

```sh
git checkout -b canary
git push origin canary

# In Jenkins, you should see the canary pipeline has kicked off. 
export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done
```

### Deploying to production

Now that our canary release was successful and we haven't heard any customer complaints, deploy to the rest of your production fleet.

```sh
git checkout master
git merge canary
git push origin master

export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done
```



