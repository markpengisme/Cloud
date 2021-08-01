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
