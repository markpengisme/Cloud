# Other Lab

[toc]

# Deploy a Web App on GKE with HTTPS Redirect using Lets Encrypt

- In this lab, you're going to deploy a containerized web app in a GKE cluster with HTTPS using a browser-trusted TLS certificate(Lets Encrypt) and NGINX to route all HTTP traffic to HTTPS. Google Cloud Endpoints is used for its ability to dynamically provision DNS entries under cloud.goog DNS domain.

- Download the source code

  ```sh
  wget https://storage.googleapis.com/spls/gsp269/gke-tls-lab-2.tar.gz
  tar zxfv gke-tls-lab-2.tar.gz
  cd gke-tls-lab
  ```

- Cloud Endpoints

  ```sh
  # Allocate a Static IP
  gcloud compute addresses create endpoints-ip --region us-central1
  gcloud compute addresses list
  
  # Replace `[MY-STATIC-IP]` with the allocated IP address.
  MY_STATIC_IP=$(gcloud compute addresses list | awk 'NR > 1 {print $2}')
  sed -i "s/\[MY-STATIC-IP\]/$MY_STATIC_IP/g" openapi.yaml
  sed -i "s/\[MY-PROJECT\]/$DEVSHELL_PROJECT_ID/g" openapi.yaml
  
  ## Deploy the Cloud Endpoints
  gcloud endpoints services deploy openapi.yaml
  ```

- Create a Kubernetes Engine Cluster

  ```sh
  ## create gke
  gcloud container clusters create cl-cluster --zone us-central1-f
  
  ## authentication 
  gcloud container clusters get-credentials cl-cluster --zone us-central1-f
  
  ## set acl
  kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole cluster-admin --user $(gcloud config get-value account)
  ```

- Add Helm Repo

  ```sh
  helm repo add stable https://charts.helm.sh/stable
  helm repo update
  
  ## deploy nginx ingress
  helm install stable/nginx-ingress --set controller.service.loadBalancerIP="$MY_STATIC_IP",rbac.create=true --generate-name
  ```

- Deploy "Hello World" App

  ```sh
  ## web app
  sed -i "s/\[MY-PROJECT\]/$DEVSHELL_PROJECT_ID/g" configmap.yaml
  kubectl apply -f configmap.yaml
  kubectl apply -f deployment.yaml
  kubectl apply -f service.yaml
  kubectl get all
  
  sed -i "s/\[MY-PROJECT\]/$DEVSHELL_PROJECT_ID/g" ingress.yaml
  kubectl apply -f ingress.yaml
  
  echo "http://api.endpoints.$DEVSHELL_PROJECT_ID.cloud.goog"
  ```

- Set Up HTTPS

  ```sh
  ## deploy cert-manager
  kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/v0.13.0/deploy/manifests/00-crds.yaml
  kubectl create namespace cert-manager
  helm repo add jetstack https://charts.jetstack.io
  helm repo update
  helm install \
    cert-manager \
    --namespace cert-manager \
    --version v0.13.0 \
    jetstack/cert-manager
  kubectl get pods --namespace cert-manager
  
  ## deploy Let's Encrypter issuer
  export QWIKLABS_USERNAME=$(whoami)@qwiklabs.net
  export EMAIL=$(echo $QWIKLABS_USERNAME | sed -e "s/_/-/g")
  cat letsencrypt-issuer.yaml | sed -e "s/email: ''/email: $EMAIL/g" | kubectl apply -f-
  
  ## Reconfigure ingress for HTTPS
  sed -i "s/\[MY-PROJECT\]/$DEVSHELL_PROJECT_ID/g" certificate.yaml
  sed -i "s/\[MY-PROJECT\]/$DEVSHELL_PROJECT_ID/g" ingress-tls.yaml
  kubectl apply -f ingress-tls.yaml
  kubectl describe ingress esp-ingress
  
  echo "http://api.endpoints.$DEVSHELL_PROJECT_ID.cloud.goog"
  ```

  

  