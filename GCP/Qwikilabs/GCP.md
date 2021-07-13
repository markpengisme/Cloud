# GCP

## Qwiklabs

### Google Cloud Essentials

#### A Tour of Qwiklabs and Google Cloud

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

#### Creating a Virtual Machine

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

