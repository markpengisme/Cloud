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
