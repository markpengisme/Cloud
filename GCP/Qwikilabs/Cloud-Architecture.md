# Cloud Architecture

## Orchestrating the Cloud with Kubernetes

### Quick Kubernetes Demo

```sh
gcloud config set compute/zone us-central1-b
gcloud container clusters create io
gsutil cp -r gs://spls/gsp021/* .
cd orchestrate-with-kubernetes/kubernetes
ls
```

```sh
kubectl create deployment nginx --image=nginx:1.10.0
kubectl get pods
kubectl expose deployment nginx --port 80 --type LoadBalancer
kubectl get services
curl http://<External IP>:80
```

### Pods

![fb02d86798243fcb.png](https://cdn.qwiklabs.com/tzvM5wFnfARnONAXX96nz8OgqOa1ihx6kCk%2BelMakfw%3D)Pods represent and hold a collection of one or more containers. Generally, if you have multiple containers with a hard dependency on each other, you package the containers inside a single pod.

Pods also have [Volumes](http://kubernetes.io/docs/user-guide/volumes/). Volumes are data disks that live as long as the pods live, and can be used by the containers in that pod. Pods provide a shared namespace for their contents which means that ==the two containers inside of our example pod can communicate with each other, and they also share the attached volumes.==

==Pods also share a network namespace. This means that there is one IP Address per pod.==

In this example there is a pod that contains the monolith and nginx containers.

### Creating Pods

```
cat pods/monolith.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: monolith
  labels:
    app: monolith
spec:
  containers:
    - name: monolith
      image: kelseyhightower/monolith:1.0.0
      args:
        - "-http=0.0.0.0:80"
        - "-health=0.0.0.0:81"
        - "-secret=secret"
      ports:
        - name: http
          containerPort: 80
        - name: health
          containerPort: 81
      resources:
        limits:
          cpu: 0.2
          memory: "10Mi"
```

```
kubectl create -f pods/monolith.yaml
kubectl get pods
kubectl describe pods monolith
```

### Interacting with Pods

```sh
kubectl port-forward monolith 10080:80
```

```sh
curl http://127.0.0.1:10080
curl http://127.0.0.1:10080/secure
curl -u user http://127.0.0.1:10080/login
password
TOKEN=$(curl http://127.0.0.1:10080/login -u user|jq -r '.token')
password
curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:10080/secure
kubectl logs monolith
kubectl logs -f monolith
```

```sh
curl http://127.0.0.1:10080
kubectl exec monolith --stdin --tty -c monolith /bin/sh
ping -c 3 google.com
exit
```

### Services

![393e02e1d49f3b37.png](https://cdn.qwiklabs.com/Jg0T%2F326ASwqeD1vAUPBWH5w1D%2F0oZn6z5mQ5MubwL8%3D)Pods aren't meant to be persistent. They can be stopped or started for many reasons - like failed liveness or readiness checks and this leads to a problem.What happens if you want to communicate with a set of Pods? When they get restarted they might have a different IP address.That's where [Services](http://kubernetes.io/docs/user-guide/services/) come in. Services provide stable endpoints for Pods.

Services use **labels** to determine what Pods they operate on. If Pods have the correct labels, they are automatically picked up and exposed by our services.

The level of access a service provides to a set of pods depends on the Service's type. Currently there are three types:

- `ClusterIP` (internal) -- the default type means that this Service is only visible inside of the cluster,
- `NodePort` gives each node in the cluster an externally accessible IP and
- `LoadBalancer` adds a load balancer from the cloud provider which forwards traffic from the service to Nodes within it.

### Creating a Service

```sh
cd ~/orchestrate-with-kubernetes/kubernetes
cat pods/secure-monolith.yaml
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-proxy-conf --from-file nginx/proxy.conf
kubectl create -f pods/secure-monolith.yaml
cat services/monolith.yaml
kubectl create -f services/monolith.yaml
```

nodeport(node) -> port(service) -> targetport(pod)

```yaml
kind: Service
apiVersion: v1
metadata:
  name: "monolith"
spec:
  selector:
    app: "monolith"
    secure: "enabled"
  ports:
    - protocol: "TCP"
      port: 443
      targetPort: 443
      nodePort: 31000
  type: NodePort
```

```sh
gcloud compute firewall-rules create allow-monolith-nodeport \
  --allow=tcp:31000
gcloud compute instances list
curl -k https://<EXTERNAL_IP>:31000
# Fail, Because pod do not have the label <secure: "enabled">, so no endpoint
```

### Adding Labels to Pods

```sh
kubectl get pods -l "app=monolith"
kubectl get pods -l "app=monolith,secure=enabled"
kubectl label pods secure-monolith 'secure=enabled'
kubectl get pods secure-monolith --show-labels
gcloud compute instances list
curl -k https://<EXTERNAL_IP>:31000
```

### Deploying Applications with Kubernetes

![f96989028fa7d280.png](https://cdn.qwiklabs.com/1UD7MTP0ZxwecE%2F64MJSNOP8QB7sU9rTI0PSv08OVz0%3D)The goal of this lab is to get you ready for scaling and managing containers in production. That's where [Deployments](http://kubernetes.io/docs/user-guide/deployments/#what-is-a-deployment) come in. Deployments are a declarative way to ensure that the number of Pods running is equal to the desired number of Pods, specified by the user.The main benefit of Deployments is in abstracting away the low level details of managing Pods. Behind the scenes Deployments use [Replica Sets](http://kubernetes.io/docs/user-guide/replicasets/) to manage starting and stopping the Pods. If Pods need to be updated or scaled, the Deployment will handle that. Deployment also handles restarting Pods if they happen to go down for some reason.



### Creating Deployments

You're going to break the monolith app into three separate pieces:

- **auth** - Generates JWT tokens for authenticated users.
- **hello** - Greet authenticated users.
- **frontend** - Routes traffic to the auth and hello services.

```
kubectl create -f deployments/auth.yaml
kubectl create -f services/auth.yaml

kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml

kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml

kubectl get services frontend
curl -k https://<EXTERNAL-IP>
```

## Continuous Delivery Pipelines with Spinnaker and Kubernetes Engine

[old note](./Google-Cloud-Solutions-Scaling-Your-Infrastructure.md)

## Multiple VPC Networks

In this lab you create several VPC networks and VM instances and test connectivity across networks. Specifically, you create two custom mode networks (**managementnet**and **privatenet**) with firewall rules and VM instances as shown in this network diagram:

![211-diagram.png](https://cdn.qwiklabs.com/OBtRY37ZCmWiHi%2FHsG8XCSGDBfsuKk3IMJVgQscsg2E%3D)

### Create custom mode VPC networks with firewall rules

- **Navigation menu** > **VPC network** > **VPC networks**

- **Create VPC Network**.

  - **Name**: `managementnet`

  - **Subnet creation mode**: **Custom**.

  - | Property         | Value (type value or select option as specified) |
    | :--------------- | :----------------------------------------------- |
    | Name             | managementsubnet-us                              |
    | Region           | us-central1                                      |
    | IP address range | 10.130.0.0/20                                    |

    **Done**

  - **EQUIVALENT COMMAND LINE**.

- **Close** > **Create**

### **Create the privatenet network**

```sh
gcloud compute networks create privatenet --subnet-mode=custom
gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24
gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west4 --range=172.20.0.0/20

gcloud compute networks list

gcloud compute networks subnets list --sort-by=NETWORK
```

### Create the firewall rules for managementnet

- **Navigation menu** > **VPC network** > **Firewall** > **Create Firewall Rule**

- | Property            | Value (type value or select option as specified)             |
  | :------------------ | :----------------------------------------------------------- |
  | Name                | managementnet-allow-icmp-ssh-rdp                             |
  | Network             | managementnet                                                |
  | Targets             | All instances in the network                                 |
  | Source filter       | IPv4 Ranges                                                  |
  | Source IPv4 ranges  | 0.0.0.0/0                                                    |
  | Protocols and ports | Specified protocols and ports, and then *check* tcp, *type:* 22, 3389; and *check*Other protocols, *type:* icmp. |

  **EQUIVALENT COMMAND LINE** > **Close** > **Create**

### Create the firewall rules for privatenet

```sh
gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0
gcloud compute firewall-rules list --sort-by=NETWORK
```

### Create VM instances

-  **Navigation menu** > **Compute Engine** > **VM instances** > **Create instance**

- 

- | Property     | Value (type value or select option as specified) |
  | :----------- | :----------------------------------------------- |
  | Name         | managementnet-us-vm                              |
  | Region       | us-central1                                      |
  | Zone         | us-central1-f                                    |
  | Series       | N1                                               |
  | Machine type | 1 vCPU (f1-micro)                                |

   **NETWORKING, DISKS, SECURITY, MANAGEMENT, SOLE-TENANCY** > **Networking** > **Network interfaces**

- | Property   | Value (type value or select option as specified) |
  | :--------- | :----------------------------------------------- |
  | Network    | managementnet                                    |
  | Subnetwork | managementsubnet-us                              |

- **Done** >  **EQUIVALENT COMMAND LINE** > **Close** > **Create**

### Create the privatenet-us-vm instance

```sh
gcloud compute instances create privatenet-us-vm --zone=us-central1-f --machine-type=n1-standard-1 --subnet=privatesubnet-us
gcloud compute instances list --sort-by=ZONE
```

- **Navigation menu** > **Compute Engine** > **VM instances** > **Column display options**: Network > **Ok**

### Explore the connectivity between VM instances

#### Ping the external IP addresses

- **Navigation menu** > **Compute Engine** > **VM instances**> **mynet-us-vm**, click **SSH**

- ```sh
  ping -c 3 <Enter mynet-eu-vm's external IP here>
  ping -c 3 <Enter managementnet-us-vm's external IP here>
  ping -c 3 <Enter privatenet-us-vm's external IP here>
  ```

  You are able to ping the external IP address of all VM instances, even though they are either in a different zone or VPC network. This confirms public access to those instances is only controlled by the **ICMP** firewall rules that you established earlier.

### Ping the internal IP addresses

- **Navigation menu** > **Compute Engine** > **VM instances**> **mynet-us-vm**, click **SSH**

- ```sh
  ping -c 3 <Enter mynet-eu-vm's internal IP here>
  ping -c 3 <Enter managementnet-us-vm's internal IP here>
  ping -c 3 <Enter privatenet-us-vm's internal IP here>
  ```

  You are able to ping the internal IP address of mynet-eu-vm because it is on the same VPC network as the source of the ping (mynet-us-vm), even though both VM instances are in separate zones, regions and continents!

  You are unable to ping the internal IP address of **managementnet-us-vm** and **privatenet-us-vm**because they are in separate VPC networks from the source of the ping (**mynet-us-vm**), even though they are all in the same zone **us-central1**.

### Create a VM instance with multiple network interfaces

- **Navigation menu** > **Compute Engine** > **VM instances** > **Create instance**

- | Property     | Value (type value or select option as specified) |
  | :----------- | :----------------------------------------------- |
  | Name         | vm-appliance                                     |
  | Region       | us-central1                                      |
  | Zone         | us-central1-f                                    |
  | Series       | N1                                               |
  | Machine type | 4 vCPUs (n1-standard-4)                          |

- **NETWORKING, DISKS, SECURITY, MANAGEMENT, SOLE-TENANCY** > **Networking** > **Network interfaces**

- | Property   | Value (type value or select option as specified) |
  | :--------- | :----------------------------------------------- |
  | Network    | privatenet                                       |
  | Subnetwork | privatesubnet-us                                 |

  **Add network interface**

  | Property   | Value (type value or select option as specified) |
  | :--------- | :----------------------------------------------- |
  | Network    | managementnet                                    |
  | Subnetwork | managementsubnet-us                              |

- **Add network interface**

  | Property   | Value (type value or select option as specified) |
  | :--------- | :----------------------------------------------- |
  | Network    | mynetwork                                        |
  | Subnetwork | mynetwork                                        |

- **Done** > **Create**

### Explore the network interface details

-  **Navigation menu**  > **Compute Engine** > **VM instances** > **vm-appliance** > **nic0**
- Verify that **nic0** is attached to **privatesubnet-us**, is assigned an internal IP address within that subnet (172.16.0.0/24), and has applicable firewall rules.
- Click **nic0** and select **nic1**.
- Verify that **nic1** is attached to **managementsubnet-us**, is assigned an internal IP address within that subnet (10.130.0.0/20), and has applicable firewall rules.
- Click **nic1** and select **nic2**.
- Verify that **nic2** is attached to **mynetwork**, is assigned an internal IP address within that subnet (10.128.0.0/20), and has applicable firewall rules.
- **Navigation menu** > **Compute Engine** > **VM instances** > **vm-appliance** > **SSH**
- `sudo ifconfig` 

### Explore the network interface connectivity

```sh
# O
ping -c 3 <Enter privatenet-us-vm's internal IP here>
# O
ping -c 3 privatenet-us-vm
# O
ping -c 3 <Enter managementnet-us-vm's internal IP here>
# O
ping -c 3 <Enter mynet-us-vm's internal IP here>
# X
ping -c 3 <Enter mynet-eu-vm's internal IP here>
ip route
```

## Troubleshooting Workloads on GKE for Site Reliability Engineers

### Scenario

Your organization has deployed a multi-tier microservices application. It is a web-based e-commerce application called "Hipster Shop", where users can browse for vintage items, add them to their cart and purchase them. Hipster Shop is composed of many microservices, written in different languages, that communicate via gRPC and REST APIs. The architecture of the deployment is optimized for learning purposes and includes modern technologies as part of the stack: Kubernetes, Istio, Cloud Operations, App Engine, gRPC, OpenTelemetry, and similar cloud-native technologies.

As a member of the Site Reliability Engineering (SRE) team, you are contacted when end users report issues viewing products and adding them to their cart. You will explore the various services deployed to determine the root cause of the issue and set up a Service Level Objective (SLO) to prevent similar incidents from occuring in the future.

### Navigating Google Kubernetes Engine (GKE) Resource Pages

- **Navigation menu** > **Kubernetes Engine** > **Clusters**.
- cloud-ops-sandbox > **Details** > **Nodes** > first Node > **Summary**
- **CPU** three dots > **View in Metrics Explorer** > Remove the filter for the `nodename`
- **How do you want to view that data?** > **Group by** to `node_name`.
- Once the filters are set the visualization will update and you will be able to view the same metrics for all of the nodes in the node pool of the `cloud-ops-sandbox` cluster.

### Accessing Operational Data Through GKE Dashboards

In this next section, you will explore how to quickly navigate to detailed operational data of various resources deployed to GKE via the GKE Dashboard.

- **Left Menu** > **Kubernetes Engine** > **Services & Ingress** > `frontend-external` IP address 

- Click on any product that is displayed on the landing page to reproduce the error reported.

- **Navigation Menu** > **Monitoring** > **Dashboards** > Select **GKE**

-  **Add Filter** > **Workloads** > **recommendationservice**

- **Workloads** > **recommendationservice** > **Deployment details**

- This view presents details on Alerts, Service Level Objectives (SLOs), Events, Metrics and Logs.

- **Metrics**

- **Logs** > **Severity** : `Error`

- Find error code: `invalid literal for int() with base 10: '5.0'`

- ```sh
  git clone --depth 1 --branch cloudskillsboost https://github.com/GoogleCloudPlatform/cloud-ops-sandbox.git
  Copied!
  cd cloud-ops-sandbox/sre-recipes
  ```

- **Navigation Menu** > **Kubernetes Engine** > **Clusters** > `cloud-ops-sandbox` three dots > **Connect** > **RUN IN CLOUD SHELL** 

- ```sh
  ./sandboxctl sre-recipes restore "recipe3"
  ```

- Check frontend again

### Proactive Monitoring with Logs-Based Metrics

To ensure that the updated `recommendationservice` code is working as expected, and to prevent future incidents from occuring again, you decide to create a logs-based metric to monitor the logs and notify SRE when similar incidents occur in the future.

-  **Navigation Menu** > **Logging** > **Logs Explorer**

- `Query results` > **Create metric**

  - Metric Type: **Counter**

  - Log metric name: **Error_Rate_SLI**

  - Filter Selection: 

    ```
    resource.labels.cluster_name="cloud-ops-sandbox" AND resource.labels.namespace_name="default" AND resource.type="k8s_container" AND labels.k8s-pod/app="recommendationservice" AND severity>=ERROR
    ```

  -  **Create Metric**.

### Creating a Service Level Objective (SLO)

After creating the logs-based metric, the SRE team decides that it will define a **Service Level Objective (SLO)** on the `recommendationservice`. You use an SLO to specify service-level objectives for performance metrics. An SLO is a measurable goal for performance over a period of time. See [Designing and using SLOs](https://cloud.google.com/service-mesh/docs/observability/design-slo) for more guidance on SLO design and the filters you will use below.

- **Navigation menu** > **Monitoring** > **Services** > `recommendationservice` > **Service details** > **+ Create SLO**
- Step1
  - Choose a metric: **Availability**
  - Request-based or windows-based: **Request Based**
  - **Continue**
- Step2
  - `Define SLI details`: the `Performance Metric` will already be assigned to measure the availability of the service based on the percentage of successful requests.
  - **Continue**
- Step3
  - `Set your service-level objective (SLO)`
  - Period type: **Calendar**
  - Period length: **Calendar month**
  - Performance Goal: **99%**
  - **Continue**
- Step4
  - **Create SLO** 
- **Current status of SLO** > **Error budget**
- The `Error budget` fraction represents the actual percentage of error budget remaining for the compliance period. In the SLO defined, there is a period of one calendar month and a performance goal of 99% or better.

### Define an Alert on the Service Level Objective (SLO)

- **Navigation menu** > **Monitoring** > **Services** > `recommendationservice` > **Current status of 1 SLO** > **Create SLO Alert** 
  - Lookback duration: 60 minutes
  - Burn rate threshold: 10
  - **Next**
  - **Manage Notification channels** > **Email**
  - Select Email just appended
  - **Next**
  - supply any information to the end user receiving the notification so that they have immediate context as to what the issue may be and ways to mitigate the problem.
  - **Save**

​	
