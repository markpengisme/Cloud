# Application Development - Python

## App Dev: Setting up a Development Environment - Python

- Create a Compute Engine Virtual Machine Instance

  - **Navigation menu** > **Compute Engine** > **VM Instances** > **Create Instance**
    - Name: `dev-instance`
    - Region: us-central1 (lowa)
    - Zone: us-central1-a
    - Machine configuration: N1
    - Identity and API access: Allow full access to all Cloud APIs
    - Firewall: Allow HTTP traffic
    - Create
  - Click **SSH**

- Install software on the VM instance

  ```sh
  sudo apt-get update
  sudo apt-get install -y git
  sudo apt-get install -y python3-setuptools python3-dev build-essential
  curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
  sudo python3 get-pip.py
  ```

- Configure the VM to Run Application Software

  ```sh
  python3 --version
  pip --version
  git clone https://github.com/GoogleCloudPlatform/training-data-analyst
  cd ~/training-data-analyst/courses/developingapps/python/devenv/
  sudo python3 server.py
  ```

  ```python
  ## Ref 
  try:
    from BaseHTTPServer import BaseHTTPRequestHandler, HTTPServer
  except ImportError:
    from http.server import BaseHTTPRequestHandler, HTTPServer
  class SimpleHTTPRequestHandler(BaseHTTPRequestHandler):
    def do_GET(self):
      self.send_response(200)
      self.send_header('Content-type','text/plain')
      self.end_headers()
      self.wfile.write(b'Hello GCP dev!')
      return
    
  def run():
    print('Server is starting...')
    server_address = ('0.0.0.0', 80)
    server = HTTPServer(server_address, SimpleHTTPRequestHandler)
    print('Started. Press Ctrl + C to stop')
    server.serve_forever()
    
  if __name__ == '__main__':
    run()
  ```

  - Click the **External IP** on **VM instances** page

  ```sh
  sudo pip3 install -r requirements.txt
  python3 list-gce-instances.py ${PROJECT_ID} --zone=us-central1-a
  ```

## App Dev: Storing Application Data in Cloud Datastore - Python

- Create a virtual environment

  ```sh
  virtualenv -p python3 vrenv
  source vrenv/bin/activate
  ```

- Prepare the Quiz application

  ```sh
  git clone https://github.com/GoogleCloudPlatform/training-data-analyst
  cd ~/training-data-analyst/courses/developingapps/python/datastore/start
  export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID
  pip install -r requirements.txt
  python run_server.py
  ```

  - **Web preview** > **Preview on port 8080**

- Examine the Quiz Application Code(Review the Flask Web application)

  - **Open Editor** > `/training-data-analyst/courses/developingapps/python/datastore/start`
  - `run.server.py`: app entrypoint, 8080 port
  - `quiz/init.py`: import routes for the web app & api
  - `quiz/webapp/questions.py` and `quiz/webapp/routes.py`: routes map URI to handlers
  - `quiz/webapp/templates`: web app user interface templates (Jinja2)
  - `quiz/api/api.py`: send json to student
  - `quiz/gcp/datastore.py`: save & load questions from Cloud Datastore

- Adding Entities to Cloud Datastore(NoSQL)

  - Create an App Engine application to provision Cloud Datastore

    ```sh
    gcloud app create --region "us-central"
    ```

  - Import and use the Python Datastore module

    ```python
    import os
    project_id = os.getenv('GCLOUD_PROJECT')
    
    from flask import current_app
    from google.cloud import datastore
    
    datastore_client = datastore.Client(project_id)
    
    """
    Returns a list of question entities for a given quiz
    - filter by quiz name, defaulting to gcp
    - no paging
    - add in the entity key as the id property 
    - if redact is true, remove the correctAnswer property from each entity
    """
    def list_entities(quiz='gcp', redact=True):
        query = datastore_client.query(kind='Question')
        query.add_filter('quiz', '=', quiz)
        results =list(query.fetch())
        for result in results:
            result['id'] = result.key.id
        if redact:
            for result in results:
                del result['correctAnswer']
        return results
    
    """
    Create and persist and entity for each question
    The Datastore key is the equivalent of a primary key in a relational database.
    There are two main ways of writing a key:
    1. Specify the kind, and let Datastore generate a unique numeric id
    2. Specify the kind and a unique string id
    """
    def save_question(question):
        key = datastore_client.key('Question')
        q_entity = datastore.Entity(key=key)
        for q_prop, q_val in question.iteritems():
            q_entity[q_prop] = q_val
        datastore_client.put(q_entity)
    ```

  - **Web preview** > **Preview on port 8080** > **Create Question**

  - **Navigation menu** > **Datastore** > **Entities**

  - **Web preview** > **Preview on port 8080** > **Take Test** > **GCP**

## App Dev: Storing Image and Video Files in Cloud Storage - Python

- Prepare the Quiz application

  ```sh
  git clone https://github.com/GoogleCloudPlatform/training-data-analyst
  cd ~/training-data-analyst/courses/developingapps/python/cloudstorage/start
  . prepare_environment.sh
  python run_server.py
  ```

  ```sh
  ## Ref
  ## run_server.py
  echo "Creating Datastore/App Engine instance"
  gcloud app create --region "us-central"
  
  echo "Creating bucket: gs://$DEVSHELL_PROJECT_ID-media"
  gsutil mb gs://$DEVSHELL_PROJECT_ID-media
  
  echo "Exporting GCLOUD_PROJECT and GCLOUD_BUCKET"
  export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID
  export GCLOUD_BUCKET=$DEVSHELL_PROJECT_ID-media
  
  echo "Creating virtual environment"
  mkdir ~/venvs
  virtualenv ~/venvs/developingapps
  source ~/venvs/developingapps/bin/activate
  
  echo "Installing Python libraries"
  pip install --upgrade pip
  pip install -r requirements.txt
  
  echo "Creating Datastore entities"
  python add_entities.py
  
  echo "Project ID: $DEVSHELL_PROJECT_ID"
  
  ```

  - **Web preview** > **Preview on port 8080**> **Create Question**

- Examine the Quiz application code

  - **Open Editor** > `/training-data-analyst/courses/developingapps/python/cloudstorage/start`
  - `quiz/webapp/templates/add.html`: template for the Create Question form.
    - `<form enctype="multipart/form-data" method="post">`
    - add two input: imageUrl(hidden) & image
  - `quiz/webapp/routes.py`: contains the routes for creating a question handler that receives the form data including image file
  - `quiz/webapp/question.py`: contains the handler that procrsses the form data extracted from routes.py
  - `quiz/gcp/storage.py`: save image file data into Cloud Storage.

- Create a Cloud Storage Bucket

  ```sh
  gsutil mb gs://$DEVSHELL_PROJECT_ID-media
  export GCLOUD_BUCKET=$DEVSHELL_PROJECT_ID-media
  ```

- Adding objects to Cloud Storage

  - Import and use the Python Cloud Storage module

    ```python
    ## quiz/gcp/storage.py
    import os
    
    project_id = os.getenv('GCLOUD_PROJECT')
    bucket_name = os.getenv('GCLOUD_BUCKET')
    
    from google.cloud import storage
    storage_client = storage.Client()
    bucket = storage_client.get_bucket(bucket_name)
    
    
    """
    Uploads a file to a given Cloud Storage bucket and returns the public url
    to the new object.
    """
    def upload_file(image_file, public):
        blob = bucket.blob(image_file.filename)
        blob.upload_from_string(
            image_file.read(),
            content_type=image_file.content_type)
        
        if public:
            blob.make_public()
            
        return blob.public_url
    ```

  - Write code to use the Cloud Storage functionality

    ```python
    ## quiz/webapp/questions.py
    from quiz.gcp import storage, datastore
    
    """
    uploads file into google cloud storage
    - upload file
    - return public_url
    """
    def upload_file(image_file, public):
        if not image_file:
            return None
        
        public_url = storage.upload_file(
           image_file, 
           public
        )
        
        return public_url
    
    """
    uploads file into google cloud storage
    - call method to upload file (public=true)
    - call datastore helper method to save question
    """
    def save_question(data, image_file):
        if image_file:
            data['imageUrl'] = unicode(upload_file(image_file, True))
        else:
            data['imageUrl'] = u''
    
        data['correctAnswer'] = int(data['correctAnswer'])
        datastore.save_question(data)
        return
    ```

  - Run the application and create a Cloud Storage object

    - [Download img file](https://storage.googleapis.com/cloud-training/quests/Google_Cloud_Storage_logo.png)

    - **Web preview** > **Preview on port 8080** > **Create Question** >**Save**

      | Form Field | Value                                                        |
      | :--------- | :----------------------------------------------------------- |
      | Author     | Mark                                                         |
      | Quiz       | Google Cloud Platform                                        |
      | Title      | Which product does this logo relate to?                      |
      | Image      | Upload the Google_Cloud_Storage_logo.png file you previously downloaded |
      | Answer 1   | App Engine                                                   |
      | Answer 2   | Cloud Storage (Select the Answer 2 radio button)             |
      | Answer 3   | Compute Engine                                               |
      | Answer 4   | Container Engine                                             |

    - **Navigation menu** > **Cloud Storage** > **Browser** > Click `<Project ID>-media` Bucket

      - Check `Google_Cloud_Storage_logo.png` exists

    - **Web preview** > **Preview on port 8080** > **Take Test** > **GCP**

      - Check the image show in the question you just added

    - Add `/api/quizzes/gcp` to app's URL

      - Check the JSON data have the question you just added

## App Dev: Adding User Authentication to your Application - Python

- Prepare the case study application

  ```sh
  git clone https://github.com/GoogleCloudPlatform/training-data-analyst
  # soft link
  ln -s ~/training-data-analyst/courses/developingapps/v1.2/python/firebase ~/firebase
  cd ~/firebase/start
  . prepare_environment.sh
  python run_server.py
  ```

  ```sh
  ## Ref
  ## prepare_environment.sh
  echo "Creating Datastore/App Engine instance"
  gcloud app create --region "us-central"
  
  echo "Creating bucket: gs://$DEVSHELL_PROJECT_ID-media"
  gsutil mb gs://$DEVSHELL_PROJECT_ID-media
  
  echo "Exporting GCLOUD_PROJECT and GCLOUD_BUCKET"
  export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID
  export GCLOUD_BUCKET=$DEVSHELL_PROJECT_ID-media
  
  echo "Creating virtual environment"
  mkdir ~/venvs
  virtualenv -p python3 ~/venvs/developingapps
  source ~/venvs/developingapps/bin/activate
  
  echo "Installing Python libraries"
  pip install --upgrade pip
  pip install -r requirements.txt
  
  echo "Creating Datastore entities"
  python add_entities.py
  
  echo "Project ID: $DEVSHELL_PROJECT_ID"
  ```

- Examine the case study application code

  - **Open Editor** > `/training_data_analyst/courses/developingapps/v1.2/python/firebase/start`
  - `quiz/webapp/static/client/index.html`: AngularJS Single Page Application
  - `quiz/webapp/static/client/app/auth/qiq-login-template.html`: AngularJS template for the Login component
  - `quiz/webapp/static/client/app/auth/qiq-login.js`: AngularJS component, allows the user to log in to the application or to navigate to a registration page

- Create a Firebase project

  - Open new tab and go to [here](https://console.firebase.google.com/) , then sign in
  - On the **Welcome to Firebase!** page, click **Add project**
  - In the **Enter your project name** dialog, select your project name.
  - Check **I accept the Firebase terms**. Click **Continue**
  - In the **Confirm Firebase billing plan** dialog, click **Confirm Plan**
  - In the **A few things to remember when adding Firebase to a Google Cloud project** dialog, click **Continue**
  - In the **Google Analytics for your Firebase project** dialog, click **Continue**
  - In the **Configure Google Analytics** dialog: Uncheck first and Check all the other(2~7), then click **Add Firebase**
  - Click **Continue**

- Configure Firebase Authentication

  - **Build > Authentication**
  - Click on **Get started**
  - In **Sign-in method** , Enable **Emall/Password**, **Enable** first, click **Save**
  - In  **Authorized Domains** , Add the quiz webapp domain, like `8080-XXXXXXXXXXX.dev`

- Integrate a client-side web application with Firebase

  - Click **Project Overview**

  - Click the web icon

  - Input nick name **quiz**, then click **Register app**

  - **Copy** Firebase SDK scripts, and open  `quiz/webapp/static/client/index.html`

  - Paste the script before the other `<script></script>` tags

  - Add the `firebase-auth.js`, the result like below

    ```html
    <!-- The core Firebase JS SDK is always required and must be listed first -->
    <script src="https://www.gstatic.com/firebasejs/8.7.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.7.1/firebase-auth.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.7.1/firebase-analytics.js"></script>
    ```

  - Save **index.html**

- Test the application

  - Return to the quiz application and refresh page
  - **Take Test** > **GCP**
  - Click **Register**, and register an account
    - You should now be logged in and redirected to the quiz
  - Click **Logout**
    - You should be logged out and redirected to the homepage

## App Dev: Developing a Backend Service - Python

- This lab concentrates on the backend service, putting together Pub/Sub, Natural Language, and Spanner services and APIs to collect and analyze feedback and scores from an online Quiz application.

- Prepare the Quiz Application

  ```sh
  git clone https://github.com/GoogleCloudPlatform/training-data-analyst
  cd ~/training-data-analyst/courses/developingapps/v1.2/python/pubsub-languageapi-spanner/start
  . prepare_web_environment.sh
  python run_server.py
  ```

  ```sh
  ## Ref
  ## run_server.py
  echo "Creating quiz-account Service Account"
  gcloud iam service-accounts create quiz-account --display-name "Quiz Account"
  gcloud iam service-accounts keys create key.json --iam-account=quiz-account@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com
  
  echo "Setting quiz-account IAM Role"
  gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member serviceAccount:quiz-account@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/owner
  
  echo "Creating Datastore/App Engine instance"
  gcloud app create --region "us-central"
  
  echo "Creating bucket: gs://$DEVSHELL_PROJECT_ID-media"
  gsutil mb gs://$DEVSHELL_PROJECT_ID-media
  
  echo "Exporting GCLOUD_PROJECT and GCLOUD_BUCKET"
  export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID
  export GCLOUD_BUCKET=$DEVSHELL_PROJECT_ID-media
  
  echo "Creating virtual environment"
  mkdir ~/venvs
  virtualenv -p python3 ~/venvs/developingapps
  source ~/venvs/developingapps/bin/activate
  
  echo "Installing Python libraries"
  pip install --upgrade pip
  pip install -r requirements.txt
  
  echo "Creating Datastore entities"
  python add_entities.py
  
  echo "Export credentials key.json"
  export GOOGLE_APPLICATION_CREDENTIALS=key.json
  
  echo "Project ID: $DEVSHELL_PROJECT_ID"
  ```

  - Second shell

  ```sh
  cd ~/training-data-analyst/courses/developingapps/v1.2/python/pubsub-languageapi-spanner/start
  . run_worker.sh
  ```

  ```sh
  ## Ref
  ## run_worker.sh
  echo "Exporting GCLOUD_PROJECT and GCLOUD_BUCKET and GOOGLE_APPLICATION_CREDENTIALS"
  export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID
  export GCLOUD_BUCKET=$DEVSHELL_PROJECT_ID-media
  export GOOGLE_APPLICATION_CREDENTIALS=key.json
  
  echo "Switching to virtual environment"
  source ~/venvs/developingapps/bin/activate
  
  echo "Starting worker"
  python -m quiz.console.worker
  ```

  - **Web preview** > **Preview on port 8080** > **Take Test** > **Places**
  - After answer the question, **Send Feedback** button not yet work

- Examine the Quiz application code

  - **Open Editor** > `/training-data-analyst/courses/developingapps/v1.2/python/pubsub-languageapi-spanner/start`
  - `quiz/gcp/pubsub.py`: This file contains a module that allows applications to publish feedback messages to a Cloud Pub/Sub topic and register a callback to receive messages from a Cloud Pub/Sub subscription.
  - `quiz/gcp/languageapi.py`: This file contains a module that allows users to send text to the Cloud Natural Language ML API and to receive the sentiment score from the API.
  - `quiz/gcp/spanner.py`: This file contains a module that allows users to save the feedback and Natural Language API response data in a Cloud Spanner database instance.
  - `quiz/api/api.py`: publishes the feedback data received from the client to Pub/Sub.
  - `quiz/console/worker.py`: This file runs as a separate console application to consume the messages delivered to a Pub/Sub subscription.

- Work with Cloud Pub/Sub

  - **Navigation menu** > **Pub/Sub** > **Topics** > **CREATE TOPIC**

    - Topic ID: feedback
    - CREATE TOPIC

  - Second Shell

    - Ctrl+c

    - Create subscription **worker-subscription**: `gcloud pubsub subscriptions create worker-subscription --topic feedback`

    - Publish & Retrieve Test

      ```sh
      gcloud pubsub topics publish feedback --message "Hello World"
      gcloud beta pubsub subscriptions pull worker-subscription --auto-ack
      ```

- Publish Messages to Cloud Pub/Sub Programmatically

  - `quiz/gcp/pubsub.py`:

    - Import and use the Python Cloud Pub/Sub module to create publisher client and topic object
    - Publish a message to Cloud Pub/Sub

    ```python
    import json
    import logging
    import os
    project_id = os.getenv('GCLOUD_PROJECT')
    
    from google.cloud import pubsub_v1
    from flask import current_app
    
    ## publisher & topic
    publisher = pubsub_v1.PublisherClient()
    topic_path = publisher.topic_path(project_id, 'feedback')
    
    
    """
    Publishes feedback info
    - jsonify feedback object
    - encode as bytestring
    - publish message
    - return result
    """
    def publish_feedback(feedback):
        payload = json.dumps(feedback, indent=2,sort_keys=True)
        data = payload.encode('utf-8')
        future = publisher.publish(topic_path, data=data)
        return future.result()
    
    ```

  - Run the application and create a Pub/Sub message

    - **Take Test** > **Places** > **Send Feedback**
    - Sencond Shell: `gcloud pubsub subscriptions pull worker-subscription --auto-ack`

- Subscribe to Cloud Pub/Sub Topics Programmatically

  - `quiz/gcp/pubsub.py`

    - Create a Cloud Pub/Sub subscription and receive messages

    ```python
    ## ...
    sub_client = pubsub_v1.SubscriberClient()
    sub_path = sub_client.subscription_path(project_id, 'worker-subscription')
    
    ## ...
    """pull_feedback
    Starts pulling messages from subscription
    - receive callback function from calling module
    - initiate the pull providing the callback function
    """
    def pull_feedback(callback):
        sub_client.subscribe(sub_path, callback=callback)
    ```

  - `quiz/console/worker.py`

    - Use the Pub/Sub subscribe functionality

    ```python
    import logging
    import sys
    import time
    import json
    from quiz.gcp import pubsub
    
    
    """
    Configure logging
    """
    logging.basicConfig(stream=sys.stdout, level=logging.INFO)
    log = logging.getLogger()
    
    """
    Receives pulled messages, analyzes and stores them
    - Acknowledge the message
    - Log receipt and contents
    """
    def pubsub_callback(message):
        message.ack()
        log.info('Message received')
        log.info(message)
    
    """
    Pulls messages and loops forever while waiting
    - initiate pull
    - loop once a minute, forever
    """
    def main():
        log.info('Worker starting...')
        ## register callback
        pubsub.pull_feedback(pubsub_callback)
        while True:
            time.sleep(60)
    
    if __name__ == '__main__':
        main()
    ```

  - Run the web and worker application and create a Pub/Sub message

    - Firsh Shell: `python run_server.py`
    - Sencond Shell: `. run_worker.sh`
    - **Take Test** > **Places** > **Send Feedback**
      - Sencon Shell will show feedback massage

- Use the Cloud Natural Language API([ref](https://cloud.google.com/natural-language/docs/reference/rest/))

  - In this section you write the code to perform sentiment analysis on the feedback text submitted by the user.

  - `quiz/gcp/languageapi.py`

    - Use the Cloud Natural Language API

    ```python
    ## import language module
    from google.cloud import language_v1
    
    ## create language api client
    lang_client = language_v1.LanguageServiceClient()
    
    """
    Returns sentiment analysis score
    - create document from passed text
    - do sentiment analysis using natural language applicable
    - return the sentiment score
    """
    def analyze(text):
        doc = language_v1.types.Document(content=text, type_='PLAIN_TEXT')
        sentiment = lang_client.analyze_sentiment(document=doc).document_sentiment
        return sentiment.score
    ```

  - `quiz/console/worker.py`

    ```python
    from quiz.gcp import pubsub, languageapi
    #...
    """
    Receives pulled messages, analyzes and stores them
    - Acknowledge the message
    - Log receipt and contents
    - convert json string
    - call helper module to do sentiment analysis
    - log sentiment score
    """
    def pubsub_callback(message):
        message.ack()
        log.info('Message received')
        log.info(message)
        data = json.loads(message.data)
        score = languageapi.analyze(str(data['feedback']))
        log.info('Score: {}'.format(score))
        data['score'] = score
    ```

  - Run the web and worker application and test the Natural Language API

    - Firsh Shell: `python run_server.py`
    - Sencond Shell: `. run_worker.sh`
    - **Take Test** > **Places** > **Send Feedback**
      - Sencon Shell will show the sentiment score

- Persist Data to Cloud Spanner

  - In this section you create a Cloud Spanner instance, database, and table. Then you write the code to persist the feedback data into the database.

  - Create a Cloud Spanner instance

    - **Navigation menu** > **Spanner** > **CREATE INSTANCE**
    - Instance name: quiz-instance
    - region: us-central1
    - Create

  - Create a Cloud Spanner database and table

    - **Instance Overview** >**quiz-instance** > **CREATE DATABASE**

    - Database Name: quiz-database

    - Schema

      ```sql
      CREATE TABLE Feedback (
          feedbackId STRING(100) NOT NULL,
          email STRING(100),
          quiz STRING(20),
          feedback STRING(MAX),
          rating INT64,
          score FLOAT64,
          timestamp INT64
      )
      PRIMARY KEY (feedbackId);
      ```

    - Create

  - Persist data into Cloud Spanner

    - `quiz/gcp/spanner.py`

      ```python
      import re
      from google.cloud import spanner
      
      """
      Get spanner management objects
      Get a reference to the Cloud Spanner quiz-instance
      Get a referent to the Cloud Spanner quiz-database
      """
      spanner_client = spanner.Client()
      instance = spanner_client.instance('quiz-instance')
      database = instance.database('quiz-database')
      
      """
      Takes an email address and reverses it (to be used as primary key)
      """
      def reverse_email(email):
          return '_'.join(list(reversed(email.replace('@','_').
                              replace('.','_').
                              split('_'))))
      
      """
      Persists feedback data into Spanner
      - create primary key value(feedback_id)
      - do a batch insert (even though it's a single record)
      """
      def save_feedback(data):
          # Create a batch object for database operations
          with database.batch() as batch:
              feedback_id = '{}_{}_{}'.format(
                          reverse_email(data['email']),
                          data['quiz'],
                          data['timestamp'])
      
              batch.insert(
                  table='feedback',
                  columns=(
                      'feedbackId',
                      'email',
                      'quiz',
                      'timestamp',
                      'rating',
                      'score',
                      'feedback'
                  ),
                  values=[
                      (
                          feedback_id,
                          data['email'],
                          data['quiz'],
                          data['timestamp'],
                          data['rating'],
                          data['score'],
                          data['feedback']
                      )
                  ]
              )
      ```

    - `quiz/console/worker.py`: Use the Cloud Spanner functionality

      ```python
      from quiz.gcp import pubsub, languageapi, spanner
      #...
      
      """
      Receives pulled messages, analyzes and stores them
      - Acknowledge the message
      - Log receipt and contents
      - convert json string
      - call helper module to do sentiment analysis
      - log sentiment score
      - call helper module to persist to spanner
      - log feedback saved
      """
      def pubsub_callback(message):
          message.ack()
          log.info('Message received')
          log.info(message)
          data = json.loads(message.data)
          score = languageapi.analyze(str(data['feedback']))
          log.info('Score: {}'.format(score))
          data['score'] = score
      
          spanner.save_feedback(data)
          log.info('Feedback saved')
      ```

  - Run the web and worker application and test Cloud Spanner

    - Firsh Shell: `python run_server.py`
    - Sencond Shell: `. run_worker.sh`
    - **Take Test** > **Places** > **Send Feedback**
      - Sencon Shell will show the feedback saved
    - **Navigation menu** > **Spanner** >  **quiz-instance > quiz-database > Query**
      - `SELECT * FROM Feedback`
      - You shoudl see the new feedback record

## App Dev: Deploying the Application into Kubernetes Engine - Python

- Prepare the Quiz Application

  ```sh
  git clone https://github.com/GoogleCloudPlatform/training-data-analyst
  ln -s ~/training-data-analyst/courses/developingapps/v1.2/python/kubernetesengine ~/kubernetesengine
  cd ~/kubernetesengine/start
  . prepare_environment.sh
  ```

  ```sh
  ## Ref.
  ## prepare_environment.sh
  
  #!/bin/bash
  
  echo "Creating quiz-account Service Account"
  gcloud iam service-accounts create quiz-account --display-name "Quiz Account"
  gcloud iam service-accounts keys create key.json --iam-account=quiz-account@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com
  
  echo "Setting quiz-account IAM Role"
  gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member serviceAccount:quiz-account@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/owner
  
  echo "Creating Datastore/App Engine instance"
  gcloud app create --region "us-central"
  
  echo "Creating bucket: gs://$DEVSHELL_PROJECT_ID-media"
  gsutil mb gs://$DEVSHELL_PROJECT_ID-media
  
  echo "Exporting GCLOUD_PROJECT and GCLOUD_BUCKET"
  export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID
  export GCLOUD_BUCKET=$DEVSHELL_PROJECT_ID-media
  
  echo "Creating virtual environment"
  mkdir ~/venvs
  virtualenv -p python3 ~/venvs/developingapps
  source ~/venvs/developingapps/bin/activate
  
  echo "Installing Python libraries"
  pip install --upgrade pip
  pip install -r requirements.txt
  
  echo "Creating Datastore entities"
  python add_entities.py
  
  echo "Export credentials key.json"
  export GOOGLE_APPLICATION_CREDENTIALS=key.json
  
  echo "Creating Cloud Pub/Sub topic"
  gcloud pubsub topics create feedback
  gcloud pubsub subscriptions create worker-subscription --topic feedback
  
  echo "Creating Cloud Spanner Instance, Database, and Table"
  gcloud spanner instances create quiz-instance --config=regional-us-central1 --description="Quiz instance" --nodes=1
  gcloud spanner databases create quiz-database --instance quiz-instance --ddl "CREATE TABLE Feedback ( feedbackId STRING(100) NOT NULL, email STRING(100), quiz STRING(20), feedback STRING(MAX), rating INT64, score FLOAT64, timestamp INT64 ) PRIMARY KEY (feedbackId);"
  
  echo "Creating Container Engine cluster"
  gcloud container clusters create quiz-cluster --zone us-central1-a --scopes cloud-platform
  gcloud container clusters get-credentials quiz-cluster --zone us-central1-a
  
  echo "Building Containers"
  gcloud container builds submit -t gcr.io/$DEVSHELL_PROJECT_ID/quiz-frontend ./frontend/
  gcloud container builds submit -t gcr.io/$DEVSHELL_PROJECT_ID/quiz-backend ./backend/
  
  echo "Deploying to Container Engine"
  sed -i -e "s/\[GCLOUD_PROJECT\]/$DEVSHELL_PROJECT_ID/g" ./frontend-deployment.yaml
  sed -i -e "s/\[GCLOUD_PROJECT\]/$DEVSHELL_PROJECT_ID/g" ./backend-deployment.yaml
  kubectl create -f ./frontend-deployment.yaml
  kubectl create -f ./backend-deployment.yaml
  kubectl create -f ./frontend-service.yaml
  
  echo "Project ID: $DEVSHELL_PROJECT_ID"
  ```

- Review the code

  - **Open Editor** > `/training_data_analyst/courses/developingapps/v1.2/python/kubernetesengine/start`
  - `frontend/`: web app
  - `backend/`: worker app(subscribes Cloud Pub/Sub and processes messages)

- Create and connect to a Kubernetes Engine Cluster

  - **Navigation menu** > **Kubernetes Engine** > **Clusters** > **CREATE**
    - Name: quiz-cluster
    - Zone: us-central1-b
    - `default Pool > Security > Access scopes`:  Allow full access to all Cloud APIs
    - Create
  - **...**(Actions) > **Connect** >  **Run in Cloud Shell**

- Build Docker Images using Container Builder

  - `frontend/Dockerfile`

    ```dockerfile
    FROM gcr.io/google_appengine/python
    
    RUN virtualenv -p python3.7 /env
    
    ENV VIRTUAL_ENV /env
    ENV PATH /env/bin:$PATH
    
    ADD requirements.txt /app/requirements.txt
    RUN pip install -r /app/requirements.txt
    
    ADD . /app
    
    CMD gunicorn -b 0.0.0.0:$PORT quiz:app
    ```

  - `backend/Dockerfile`

    ```dockerfile
    FROM gcr.io/google_appengine/python
    
    RUN virtualenv -p python3.7 /env
    
    ENV VIRTUAL_ENV /env
    ENV PATH /env/bin:$PATH
    
    ADD requirements.txt /app/requirements.txt
    RUN pip install -r /app/requirements.txt
    
    ADD . /app
    
    CMD python -m quiz.console.worker
    ```

  - Cloud Shell

    ```sh
    cd ~/kubernetesengine/start
    ##  build the frontend Docker image:
    gcloud builds submit -t gcr.io/$DEVSHELL_PROJECT_ID/quiz-frontend ./frontend/
    ##  build the backend Docker image:
    gcloud builds submit -t gcr.io/$DEVSHELL_PROJECT_ID/quiz-backend ./backend/
    ```

    - **Navigation** > **Container Registry** > Check two images

- Create Kubernetes Deployment and Service Resources

  - Print env varaibles by Cloud shell

    ```sh
    echo [GCLOUD_PROJECT] = $GCLOUD_PROJECT
    echo [GCLOUD_BUCKET] = $GCLOUD_BUCKET
    echo [FRONTEND_IMAGE_IDENTIFIER] = gcr.io/${GCLOUD_PROJECT}/quiz-frontend
    echo [BACKEND_IMAGE_IDENTIFIER] = gcr.io/${GCLOUD_PROJECT}/quiz-backend
    ```

  - `frontend-deployment.yaml`: replace the placeholder, [ref source code](https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/courses/developingapps/v1.2/python/kubernetesengine/end/frontend-deployment.yaml)

  - `backend-deployment.yaml`: replace the placeholder, [ref. source code](https://github.com/GoogleCloudPlatform/training-data-analyst/blob/master/courses/developingapps/v1.2/python/kubernetesengine/end/backend-deployment.yaml)

  - Execute the Deployment and Service Files

    ```sh
    kubectl create -f ./frontend-deployment.yaml
    kubectl create -f ./backend-deployment.yaml
    kubectl create -f ./frontend-service.yaml
    ```

  - Review the deployed resources

    - **Navigation menu** > **Kubernetes Engine** > **Workloads**
      - Status OK
      - Pods: 2/2 & 3/3
    - **quiz-frontend** > **Exposing services** > **Click Endpoints**
    - Test the Quiz Application
