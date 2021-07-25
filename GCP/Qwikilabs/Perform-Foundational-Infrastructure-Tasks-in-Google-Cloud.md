# Perform Foundational Infrastructure Tasks in Google Cloud

[toc]

## Cloud Storage: Qwik Start - Cloud Console

- Cloud Storage allows world-wide storage and retrieval of any amount of data at any time. You can use Cloud Storage for a range of scenarios including serving website content, storing data for archival and disaster recovery, or distributing large data objects to users via direct download.
- Create a Bucket
  - **Navigation menu** > **Cloud Storage** > **Browser** > **Create a bucket**
    - Name: Project ID
    - Region: us-east1(South Carolina)
    - Default sotrage class: Standard
    - Access control: Uniform

- Upload an object into the Bucket
  - **Click Bucket Name** > **Upload files**
  - **Click ...** > **Rename**

- Share an Object Publicly
  - **Permissions**> **Public Access** > **Remove Public Access Prevention** > **Confirm**
  - **Permissions**> **Members** > **ADD** > **allUsers** > **Cloud Storage** > **Storage Object Viewer** > **Save**
  - **Object** > **Public access** > **Copy URL**
- Create Folders
- Delete Folders
- Download

## Cloud Storage: Qwik Start - CLI/SDK

- Create a Bucket

  - **Navigation menu** > **Cloud Storage** > **Browser** > **Create a bucket**
    - Name: Project ID
    - MultiRegion: us
    - Default sotrage class: Standard
    - **Uncheck** *Enforce public access prevention on this bucket* checkbox
    - Access control: Fine-grained

- Upload an object into your bucket

  ```sh
  wget --output-document ada.jpg https://upload.wikimedia.org/wikipedia/commons/thumb/a/a4/Ada_Lovelace_portrait.jpg/800px-Ada_Lovelace_portrait.jpg
  gsutil cp ada.jpg gs://${YOUR_BUCKET_NAME}rm ada.jpg
  ```

- Download an object from your bucket

  ```sh
  gsutil cp -r gs://${YOUR_BUCKET_NAME}/ada.jpg .
  ```

- Copy an object to a folder in the bucket

  ```sh
  gsutil cp gs://${YOUR_BUCKET_NAME}/ada.jpg gs://${YOUR_BUCKET_NAME}/image-folder/
  ```

- List contents of a bucket or folder

  ```sh
  gsutil ls gs://${YOUR_BUCKET_NAME}
  gsutil ls -l gs://YOUR-BUCKET-NAME/ada.jpg
  ```

- Make your object publicly accessible

  ```sh
  gsutil acl ch -u AllUsers:R gs://${YOUR_BUCKET_NAME}/ada.jpg
  ```

- Remove public access

  ```sh
  gsutil acl ch -d AllUsers gs://${YOUR_BUCKET_NAME}/ada.jpg
  ```

- Delete objects

  ```sh
  gsutil rm gs://${YOUR_BUCKET_NAME}/ada.jpg
  ```

## Cloud IAM: Qwik Start

- Google Cloud's Identity and Access Management (IAM) service lets you create and manage permissions for Google Cloud resources.

- [Google Cloud roles documentation](https://cloud.google.com/iam/docs/understanding-roles#primitive_roles)

- The IAM console and project level roles(Username1)

  - **Navigation menu** > **IAM & Admin** > **IAM** > **ADD** > **Select a role**
  - You should see Browser, Editor, Owner, and Viewer roles. These four are known as *primitive roles* in Google Cloud.
  - Cancel

- Explore editor roles(Username 2)

  - **Navigation menu** > **IAM & Admin** > **IAM**
  - You should see Username 2 has the "Viewer" role granted to the project
  - And the **ADD** button at the top is grayed out

- Prepare a resource for access testing(Username1)

  - **Navigation menu** > **Cloud Storage** > **Browser** > **Create a bucket**
    - Name: Project ID
    - MultiRegion: us
  - Upload any txt file
  - Reanme to `sample.txt`

- Verify project viewer access(Username 2)

  - **Navigation menu** > **Cloud Storage** > **Browser**
  - Username 2 has the "Viewer" role prescribed which allows them read-only actions that do not affect state

- Remove project access(Username 1)

  - **Navigation menu** > **IAM & Admin** > **IAM**
  - Remove Project Viewer access for **Username 2**(Pencil icon > Trashcan icon), then click **Save**

- Verify that Username 2 has lost access(Username2)

- Add Storage permissions(Usernam1)

  - **Navigation menu** > **IAM & Admin** > **IAM** >**ADD**
     - New members: Username2
     - Select a role: **Cloud Storage** > **Storage Object Viewer**
     - Save

- Verify access(Username2)

  ```sh
  gsutil ls gs://${YOUR_BUCKET_NAME}
  ```

## Cloud Monitoring: Qwik Start

- Cloud Monitoring provides visibility into the performance, uptime, and overall health of cloud-powered applications.

- Create a Compute Engine instance

  - **Navigation menu** > **Compute Engine** > **VM instances** > **Create instance**

    | Name         | lamp-1-vm                |
    | ------------ | ------------------------ |
    | Region       | us-central1 (Iowa)       |
    | Zone         | us-central1-a            |
    | Series       | N1                       |
    | Machine type | n1-standard-2            |
    | Firewall     | check Allow HTTP traffic |

- Add Apache2 HTTP Server to your instance

  - **SSH**

    ```sh
    sudo apt-get update
    sudo apt-get install -y apache2 php7.0
    sudo service apache2 restart
    ```

  - Click `lamp-1-vm`'s `External IP`

- Create a Monitoring workspace

  - **Navigation menu** > **Monitoring**

- Install the Monitoring and Logging agents

  - **SSH**

    ```sh
    ## install monitor agent
    curl -sSO https://dl.google.com/cloudagents/add-monitoring-agent-repo.sh
    sudo bash add-monitoring-agent-repo.sh
    sudo apt-get update
    sudo apt-get install -y stackdriver-agent
    
    ## install logging agent
    curl -sSO https://dl.google.com/cloudagents/add-logging-agent-repo.sh
    sudo bash add-logging-agent-repo.sh
    sudo apt-get update
    sudo apt-get install -y google-fluentd
    ```

- Create an uptime check

  - **Navigation menu** > **Monitoring** > **Uptime checks** >  **Create Uptime Check**
  - Title: Lamp Uptime Check
    - Protocol: HTTP
  - Resource Type: Instance
    - Applies to: Single, lamp-1-vm
  - Path: leave at default
    - Duration: 1 min
  - Test > Create

- Create an alerting policy

  - **Navigation menu** > **Monitoring** > **Alerting** > **Create Policy**
  - Add condition
    - Target
      - Resource Type: VM Instance (gce_instance)
      - Metric: Network traffic (gce_instance+1) `agent.googleapis.com/interface/traffic`
    - Configuration
      - Condition: is above
      - Threshold: 500
      - For: 1 minute
    - ADD > Next
  - **Notification Channels** > **Manage Notification Channels** > **EMAIL** > **ADD NEW**
    - Email: personal email
    - Display Name: random
  - **Notification Channels** > **Refresh button** > select your display name > OK > Next
  - Alert Name: Inbound Traffic Alert
  - Documentation: Alert Test
  - Save

- Create a dashboard and chart

  - **Navigation menu** > **Monitoring** > **Dashboard** > **Create Dashboard**
  - Name: Cloud Monitoring LAMP Qwik Start Dashboard
  - ADD CHART > Line
    - CPU Load
    - VM Instance
    - CPU Load(1m)
  - ADD CHART > Line
    - Received Packets
    - VM Instance
    - Received Packets
  - Refresh

- View your logs

  - **Navigation menu** > **Logging** > **Logs Explorer**
    - **Resource** > **VM Instance** > **lamp-1-vm** > **ADD**
    - Stream logs
    - Open new tab , and check out what happens when you start and stop the VM instance.

- Check the uptime check results and triggered alerts

  - **Navigation menu** > **Monitoring** > **Uptime checks**.
  - This view provides a list of all active uptime checks, and the status of each in different locations.
  - Click Name to see details
  - **Navigation menu** > **Monitoring** > **Alert**
  - You see incidents and events listed in the Alerting window.
  - Check email you should see Cloud Monitoring Alerts.
