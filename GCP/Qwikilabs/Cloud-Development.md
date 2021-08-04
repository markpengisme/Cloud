# Cloud Development

## App Dev: Setting up a Development Environment - Python

[old note](./Application-Development-Python.md#App-Dev:-Setting-up-a-Development-Environment---Python)

## App Dev: Storing Application Data in Cloud Datastore - Python

[old note](./Application-Development-Python.md#App-Dev:-Storing-Application-Data-in-Cloud-Datastore---Python)

## App Dev: Storing Image and Video Files in Cloud Storage - Python

[old note](./Application-Development-Python.md#App-Dev:-Storing-Image-and-Video-Files-in-Cloud-Storage---Python)

## App Dev: Adding User Authentication to your Application - Python

[old note](./Application-Development-Python.md#App-Dev:-Adding-User-Authentication-to your-Application---Python)

## App Dev: Developing a Backend Service - Python

[old note](./Application-Development-Python.md#App-Dev:-Developing-a-Backend-Service---Python)

## App Dev: Deploying the Application into Kubernetes Engine - Python

[old note](./Application-Development-Python.md#App-Dev:-Deploying-the-Application-into Kubernetes-Engine---Python)

## App Dev - Deploying the Application into App Engine Flexible Environment - Java

- An App Engine app is made up of a single application resource that consists of one or more *services*. Each service can be configured to use different runtimes and to operate with different performance settings. Within each service, you deploy *versions* of that service. Each version then runs within one or more *instances*, depending on how much traffic you configured it to handle.([ref](https://cloud.google.com/appengine/docs/flexible/python/an-overview-of-app-engine))

- Preparing the Case Study Application

  ```sh
  git clone https://github.com/GoogleCloudPlatform/training-data-analyst
  cd ~/training-data-analyst/courses/developingapps/java/appengine/start
  . prepare_environment.sh
  ```

- Review the code

  - **Open Editor** > `training-data-analyst/courses/developingapps/java/appengine/start`

- Preparing Application Code for App Engine Flexible Environment Deployment

  - Create the app.yaml file for the frontend: `src/main/appengine/app.yaml`

  - Replace the [GCLOUD_BUCKET]

    ```yaml
    runtime: java
    env: flex
    runtime_config:
      jdk: openjdk8
    handlers:
    - url: /.*
      script: this field is required, but ignored
    manual_scaling:
      instances: 1
    resources:
      cpu: 1
      memory_gb: 3.75
      disk_size_gb: 10
    env_variables:
      GCLOUD_BUCKET: [GCLOUD_BUCKET]
    ```

  - Deploy the quiz application to App Engine flexible environment

    ```sh
    ## need 10 mins
    mvn clean compile appengine:deploy
    ```

  - **Navigation menu** > **App Engine** > **Dashboard** > **top-right app link**

- Updating an App Engine Flexible Environment Application

  - `src/main/resources/static/index.html`: Add several `!` to test
    - `<h1>Welcome to the Quite Interesting Quiz!!!!!</h1>`

  ```sh
  ## the previous version will continue to receive traffic.
  mvn clean compile appengine:deploy \
  -Dapp.deploy.stopPreviousVersion=False \
  -Dapp.deploy.promote=False
  ```

  - **Navigation menu** > **App Engine** > **Dashboard** > **top-right app link**

    - You should see that your application still displays the old title.

  - **Navigation menu** > **App Engine** > **Versions**

    - Click on both version links to see the new and old version of the quiz application

  - **Navigation menu** > **App Engine** > **Versions** >

    - Select two versions
    - Click **Split traffic** button
    - Split traffic by **Random**
    - Split to 50%/50% of traffic to two versions
    - Save

  - **Navigation menu** > **App Engine** > **Dashboard** > set **Version**=**All Version** > **app link**

  - Refresh the homepage a few times, You should see that the homepage displays the old version approximately half the time, and the new version half the time.

    > In real-world scenarios, you might start by delivering small amounts of traffic to the new version in a canary release, and would use either a cookie or IP address to ensure that a client viewed a single consistent version of the applicatio

## Cloud Monitoring: Qwik Start

[old note](./Perform-Foundational-Infrastructure-Tasks-in-Google-Cloud.md#Cloud-Monitoring:-Qwik-Start)

## Cloud Profiler: Qwik Start

- Cloud Profiler is a statistical, low-overhead profiler that continuously gathers CPU usage and memory-allocation information from your production applications. It attributes that information to the application's source code, helping you identify the parts of the application consuming the most resources, and otherwise illuminating the performance characteristics of the code.

- **APIs & services** > **Dashboard** > **Enable Cloud Profiler API** > **Stackdriver Profiler API** > **Enable**

- Get a program to profile

  - `go get -u github.com/GoogleCloudPlatform/golang-samples/profiler/profiler_quickstart`

- Profile the code

  - `profiler_quickstart`: This program is designed to to load the CPU as it runs, and configured to use Cloud Profiler. Cloud Profiler collects profiling data from the program as it runs and periodically saves it.

- Start the Profiler interface: **Navigation menu** > **Profiler**，The interface is divided into two general areas:

  - A control area for selecting the data to visualize.
  - A flame-graph representation of the selected data.

- Another tutorials: <https://cloud.google.com/profiler/docs/quickstart-go-app?hl=zh-tw>
