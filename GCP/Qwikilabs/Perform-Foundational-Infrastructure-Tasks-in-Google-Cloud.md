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
  - **Notification Channels** > **Refresh button** > Select your display name > OK > Next
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

## Cloud Functions: Qwik Start - Console

- Google Cloud Functions is a lightweight, event-based, asynchronous compute solution that allows you to create small, single-purpose functions that respond to cloud events without the need to manage a server or a runtime environment

  - Cloud events are things that happen in your cloud environment.
  - You create a response to an event with a trigger.
  - Binding a function to a trigger allows you to capture and act on events
  - [Events and Triggers](https://cloud.google.com/functions/docs/concepts/events-triggers)
  - Use Case: Data Processing / ETL, Webhooks, Lightweight API, Mobile backend, IoT

- Create a function

  - **Navigation menu** > **Cloud Functions** > **Create Function**

    | **Field**                                                    | **Value**                          |
    | ------------------------------------------------------------ | ---------------------------------- |
    | Function name                                                | GCFunction                         |
    | Trigger                                                      | Select **HTTP** and click **Save** |
    | Memory allocated (In Runtime, Build and Connections Settings) | Keep it default and click **Next** |

- Deploy the function

  - use default helloworld func

    ```js
    /**
     * Responds to any HTTP request.
     *
     * @param {!express:Request} req HTTP request context.
     * @param {!express:Response} res HTTP response context.
     */
    exports.helloWorld = (req, res) => {
      let message = req.query.message || req.body.message || 'Hello World!';
      res.status(200).send(message);
    };
    ```

- Test the function

  - **...** > **Test function**
  - `{"message":"Hello World!"}`
  - **Test the function**
  - See the **Output** field
  - See the **Logs** field: 200 status code

- View logs

  - go back > **...** > **View Logs**
  - You can see 2 logs about the GCFunction

## Cloud Functions: Qwik Start - Command Line

- Create a function

  - you're going to create a simple function named helloWorld. This function writes a message to the Cloud Functions logs.
  - For this lab the cloud function event is a cloud pub/sub topic event. A pub/sub is a messaging service where the senders of messages are decoupled from the receivers of messages. When a message is sent or posted, a subscription is required for a receiver to be alerted and receive the message. Ref.[Google Cloud Pub/Sub: A Google-Scale Messaging Service](https://cloud.google.com/pubsub/architecture).

  ```sh
  mkdir gcf_hello_world
  cd gcf_hello_world
  vim index.js
  ```

  ```js
  /**
  * Background Cloud Function to be triggered by Pub/Sub.
  * This function is exported by index.js, and executed when
  * the trigger topic receives a message.
  *
  * @param {object} data The event payload.
  * @param {object} context The event metadata.
  */
  exports.helloWorld = (data, context) => {
  const pubSubMessage = data;
  const name = pubSubMessage.data
      ? Buffer.from(pubSubMessage.data, 'base64').toString() : "Hello World";
  
  console.log(`My Cloud Function: ${name}`);
  };
  ```

- Create a cloud storage bucket

  ```sh
  gsutil mb -p ${PROJECT_ID} gs://${BUCKET_NAME}
  ```

- Deploy function

  ```sh
  gcloud functions deploy helloWorld \
      --stage-bucket ${BUCKET_NAME} \
      --trigger-topic hello_world \
      --runtime nodejs8  
      
  ## Check active
  gcloud functions describe helloWorld
  ```

- Test the function

  ```sh
  DATA=$(printf 'Hello World!'|base64) && gcloud functions call helloWorld --data '{"data":"'$DATA'"}'
  ```

- View logs

  ```sh
  ## need around 10 mins
  ## Alternative way: Logging > Logs Explorer
  gcloud functions logs read helloWorld
  ```

## Google Cloud Pub/Sub: Qwik Start - Console

- Google Cloud Pub/Sub is a messaging service for exchanging event data among applications and services. A producer of data publishes messages to a Cloud Pub/Sub topic. A consumer creates a subscription to that topic. Subscribers either pull messages from a subscription or are configured as webhooks for push subscriptions. Every subscriber must acknowledge each message within a configurable window of time.

- Setting up Pub/Sub

  - **Navigation menu** > **Pub/Sub** > **Topics** > **Create topic**
  - Topic ID:MyTopic
  - uncheck the default subscription
  - CREATE TOPIC

- Add a subscription

  - Click Topic **...** > **Create subscription**

  - Subscription ID: MySub
  - Delivery type: Pull
  - CREATE

- Publish a message to the topic

  - Click **Topic ID**, see the topic details
  - PUBLISH MESSAGE
  - Message: Hello World
  - PUBLISH

- View the message

  - VIEW MESSAGES
  - Select Mysub
  - PULL

## Google Cloud Pub/Sub: Qwik Start - Command Line

- The Pub/Sub basics: a producer publishes messages to a topic and a consumer creates a subscription to a topic to receive messages from it.

- Pub/Sub topics & Pub/Sub subscriptions

  ```sh
  ## create topics and list them
  gcloud pubsub topics create myTopic
  gcloud pubsub topics create Test1
  gcloud pubsub topics create Test2
  gcloud pubsub topics list
  gcloud pubsub topics delete Test1
  gcloud pubsub topics delete Test2
  gcloud pubsub topics list
  
  ## create subscriptions to the topic and list them
  gcloud pubsub subscriptions create --topic myTopic mySubscription
  gcloud pubsub subscriptions create --topic myTopic Test1
  gcloud pubsub subscriptions create --topic myTopic Test2
  gcloud pubsub topics list-subscriptions myTopic
  gcloud pubsub subscriptions delete Test1
  gcloud pubsub subscriptions delete Test2
  gcloud pubsub topics list-subscriptions myTopic
  ```

- Pub/Sub Publishing and Pulling the Message

  - Using the pull command without any flags will output only one message, even if you are subscribed to a topic that has more held in it.

  ```sh
  ## publish messages
  gcloud pubsub topics publish myTopic --message "Hello"
  gcloud pubsub topics publish myTopic --message "Publisher's name is <YOUR NAME>"
  gcloud pubsub topics publish myTopic --message "Publisher likes to eat <FOOD>"
  gcloud pubsub topics publish myTopic --message "Publisher thinks Pub/Sub is awesome"
  
  ## pull by subscription
  gcloud pubsub subscriptions pull mySubscription --auto-ack
  gcloud pubsub subscriptions pull mySubscription --auto-ack
  gcloud pubsub subscriptions pull mySubscription --auto-ack
  gcloud pubsub subscriptions pull mySubscription --auto-ack
  
  ## publish messages again
  gcloud pubsub topics publish myTopic --message "Publisher is starting to get the hang of Pub/Sub"
  gcloud pubsub topics publish myTopic --message "Publisher wonders if all messages will be pulled"
  gcloud pubsub topics publish myTopic --message "Publisher will have to test to find out"
  
  ## pull all
  gcloud pubsub subscriptions pull mySubscription --auto-ack --limit=3
  ```

## Google Cloud Pub/Sub: Qwik Start - Python

- Create a virtual environment

  ```sh
  sudo apt-get update
  sudo apt-get install -y virtualenv
  virtualenv -p python3 venv
  source venv/bin/activate
  ```

- Install the client library

  ```sh
  pip install --upgrade google-cloud-pubsub
  git clone https://github.com/googleapis/python-pubsub.gitcd python-pubsub/samples/snippets
  ```

- Create a topic & subsciption

  ```sh
  export GLOBAL_CLOUD_PROJECT=GCP Project ID
  cat publisher.py
  python publisher.py -h
  
  ## create a topic
  python publisher.py $GLOBAL_CLOUD_PROJECT create MyTopic
  
  ## list
  python publisher.py $GLOBAL_CLOUD_PROJECT list
  cat subscriber.py
  python subscriber.py -h
  
  ## Create a subscription
  python subscriber.py $GLOBAL_CLOUD_PROJECT create MyTopic MySub
  
  ## list
  python subscriber.py $GLOBAL_CLOUD_PROJECT list-in-topic MyTopic
  ```

- Publish and Pull messages

  ```sh
  ## publish messages
  python publisher.py $GLOBAL_CLOUD_PROJECT publish MyTopic
  ## pull messages
  python subscriber.py $GLOBAL_CLOUD_PROJECT receive MySub
  ```
