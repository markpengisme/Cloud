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
