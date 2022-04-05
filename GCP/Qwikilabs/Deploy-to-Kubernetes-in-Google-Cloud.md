# Deploy to Kubernetes in Google Cloud

## Introduction to Docker

Docker is an open platform for developing, shipping, and running applications. With Docker, you can separate your applications from your infrastructure and treat your infrastructure like a managed application. Docker helps you ship code faster, test faster, deploy faster, and shorten the cycle between writing code and running code.

Docker does this by combining kernel containerization features with workflows and tooling that helps you manage and deploy your applications.

Docker containers can be directly used in Kubernetes, which allows them to be run in the Kubernetes Engine with ease. After learning the essentials of Docker, you will have the skillset to start developing Kubernetes and containerized applications.

### Hello World

```sh
docker run hello-world
docker images
docker run hello-world
docker ps
docker ps -a
```

### Build

```sh
mkdir test && cd test
cat > Dockerfile <<EOF
# Use an official Node runtime as the parent image
FROM node:6
# Set the working directory in the container to /app
WORKDIR /app
# Copy the current directory contents into the container at /app
ADD . /app
# Make the container's port 80 available to the outside world
EXPOSE 80
# Run app.js using node when the container launches
CMD ["node", "app.js"]
EOF
```

```sh
cat > app.js <<EOF
const http = require('http');
const hostname = '0.0.0.0';
const port = 80;
const server = http.createServer((req, res) => {
    res.statusCode = 200;
      res.setHeader('Content-Type', 'text/plain');
        res.end('Hello World\n');
});
server.listen(port, hostname, () => {
    console.log('Server running at http://%s:%s/', hostname, port);
});
process.on('SIGINT', function() {
    console.log('Caught interrupt signal and will exit');
    process.exit();
});
EOF
```

```sh
docker build -t node-app:0.1 .
docker images
```

### Run

```sh
docker run -p 4000:80 --name my-app node-app:0.1
# shell 2
curl http://localhost:4000

docker stop my-app && docker rm my-app
docker run -p 4000:80 --name my-app -d node-app:0.1
docker ps
docker logs [container_id]
cd test
```

```js
// app.js
....
const server = http.createServer((req, res) => {
    res.statusCode = 200;
      res.setHeader('Content-Type', 'text/plain');
        res.end('Welcome to Cloud\n');
});
....
```

```
docker build -t node-app:0.2 .
docker run -p 8080:80 --name my-app-2 -d node-app:0.2
docker ps
```

```
curl http://localhost:8080
curl http://localhost:4000
```

### Debug

```sh
docker logs -f [container_id]
docker exec -it [container_id] bash
ls
exit
docker inspect [container_id]
docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' [container_id]
```

### Publish

To push images to your private registry hosted by gcr, you need to tag the images with a registry name. The format is `[hostname]/[project-id]/[image]:[tag]`.

For gcr:

- `[hostname]`= gcr.io
- `[project-id]`= your project's ID
- `[image]`= your image name
- `[tag]`= any string tag of your choice. If unspecified, it defaults to "latest".

```sh
gcloud config list project
docker tag node-app:0.2 gcr.io/[project-id]/node-app:0.2
docker images
docker push gcr.io/[project-id]/node-app:0.2
```

-  **Navigation menu** > **Container Registry** > **node-app**
- <http://gcr.io/[project-id]/node-app>

### Remove and Pull

```sh
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
docker ps -a

docker rmi node-app:0.2 gcr.io/[project-id]/node-app node-app:0.1
docker rmi node:6
docker rmi $(docker images -aq) # remove remaining images
docker images
```

```sh
docker pull gcr.io/[project-id]/node-app:0.2
docker run -p 4000:80 -d gcr.io/[project-id]/node-app:0.2
curl http://localhost:4000
```

## Kubernetes Engine: Qwik Start

[old note]

## Orchestrating the Cloud with Kubernetes

[old note]

