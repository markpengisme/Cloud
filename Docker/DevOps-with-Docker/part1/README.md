# 1.1: Getting started

```txt
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS                     PORTS     NAMES
89fa9c68e67f   nginx     "/docker-entrypoint.…"   58 seconds ago       Exited (0) 5 seconds ago             hopeful_poincare
41168bfcf045   nginx     "/docker-entrypoint.…"   59 seconds ago       Exited (0) 6 seconds ago             stoic_aryabhata
fcbb7434f81a   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute          80/tcp    infallible_mestorf
```

# 1.2: Cleanup

```
MarkPeng:DevOps-with-Docker/ (main✗) $ docker ps                                                        
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
MarkPeng:DevOps-with-Docker/ (main✗) $ docker images                                                    
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

# 1.3: Secret message

```sh
MarkPeng:DevOps-with-Docker/ (main✗) $ docker run -d --name secret devopsdockeruh/simple-web-service:ubuntu
43d2dd6f7f080c19c1eadbfa263449a88964c457a841d4b60a6ef6173c878d42
MarkPeng:DevOps-with-Docker/ (main✗) $ docker exec -it secret sh -c "tail -f ./text.log"                        

Secret message is: 'You can find the source code here: https://github.com/docker-hy'
```

# 1.4: Missing dependencies

```sh
docker run -d -it --rm --name read-website ubuntu sh -c 'apt update; apt install -y curl; echo "Input website:"; read website; echo "Searching.."; sleep 1; curl http://$website;'
docker attach read-website
```

# 1.5: Sizes of images

```sh
docker pull devopsdockeruh/simple-web-service:ubuntu
docker pull devopsdockeruh/simple-web-service:alpine

MarkPeng:DevOps-with-Docker/ (main✗) $ docker images devopsdockeruh/simple-web-service                          
REPOSITORY                          TAG       IMAGE ID       CREATED        SIZE
devopsdockeruh/simple-web-service   ubuntu    4e3362e907d5   8 months ago   83MB
devopsdockeruh/simple-web-service   alpine    fd312adc88e0   8 months ago   15.7MB
```

# 1.6: Hello Docker Hub

```sh
docker run -it devopsdockeruh/pull_exercise

Give me the password: basics
You found the correct password. Secret message is:
"This is the secret message"
```

# 1.7: Two line Dockerfile

```dockerfile
FROM devopsdockeruh/simple-web-service:alpine
CMD ["server"]
```

```sh
docker build . -t web-server
docker run web-server
```

# 1.8: Image for script

```sh
## start.sh
#!/bin/sh
echo "Input website:";
read website;
echo "Searching..";
sleep 1;
curl http://$website;
```

```
FROM ubuntu:18.04 

WORKDIR /mydir 
RUN apt-get update && apt-get install -y curl
COPY start.sh ./start.sh
CMD ["/bin/bash","start.sh"]
```

```sh
docker build . -t curler
docker run -it curler
helsinki.fi
```

# 1.9: Volumes

```sh
touch /tmp/text.log
docker run -v /tmp/text.log:/usr/src/app/text.log devopsdockeruh/simple-web-service
```

# 1.10: Ports open

```sh
docker run -p 80:8080 devopsdockeruh/simple-web-service -c "server"
```

# 1.11: Spring

```dockerfile
FROM openjdk:8

WORKDIR /usr/src/app

RUN apt update && \
    apt install -y git
  
RUN git clone https://github.com/docker-hy/material-applications.git
RUN cd material-applications/spring-example-project && ./mvnw package

EXPOSE 8080
CMD ["/usr/local/openjdk-8/bin/java", "-jar", "/usr/src/app/material-applications/spring-example-project/target/docker-example-1.1.3.jar"]
```

```sh
docker build -t simple-button .
docker run -p 80:8080 simple-button
```

# 1.12: Hello, frontend!

```dockerfile
FROM node:fermium-alpine

WORKDIR /usr/src/app

RUN apk update && \
    apk add git && \
    git clone https://github.com/docker-hy/material-applications.git && \
    mv material-applications/example-frontend/* ./ &&\
    rm -rf material-applications
RUN npm install && node -v && npm -v && \
		npm run build && \
		npm install -g serve
		
EXPOSE 5000		

CMD serve -s -l 5000 build
```

```sh
docker build -t example-frontend .
docker run -d -p 5000:5000 example-frontend
```

http://localhost:5000

# 1.13: Hello, backend!

```dockerfile
FROM golang:1.16-alpine

WORKDIR /usr/src/app

RUN apk update && \
    apk add git build-base && \
    git clone https://github.com/docker-hy/material-applications.git && \
    mv material-applications/example-backend/* ./ && \
    rm -rf material-applications
RUN go build
RUN go test ./...

EXPOSE 8080

CMD ./server
```

```sh
docker build -t example-backend .
docker run -d -p 8080:8080 example-backend
```

 http://localhost:8080/ping

# 1.14: Environment

```dockerfile
# frontend
FROM node:fermium-alpine

WORKDIR /usr/src/app

RUN apk update && \
    apk add git && \
    git clone https://github.com/docker-hy/material-applications.git && \
    mv material-applications/example-frontend/* ./ &&\
    rm -rf material-applications

ENV REACT_APP_BACKEND_URL=http://localhost:8080/
RUN npm install && node -v && npm -v && \
		npm run build && \
		npm install -g serve

EXPOSE 5000

CMD serve -s -l 5000 build
```

```dockerfile
# backend
FROM golang:1.16-alpine

WORKDIR /usr/src/app

RUN apk update && \
    apk add git build-base && \
    git clone https://github.com/docker-hy/material-applications.git && \
    mv material-applications/example-backend/* ./ && \
    rm -rf material-applications

ENV REQUEST_ORIGIN http://localhost:5000
RUN go build
RUN go test ./...

EXPOSE 8080

CMD ./server
```

```sh
docker build ./frontend -t example-frontend
docker build ./backend -t example-backend
docker run -d -p 8080:8080 example-backend
docker run -d -p 5000:5000 example-frontend
```

http://localhost:5000

# 1.15: Homework

```dockerfile
FROM ubuntu:20.04

WORKDIR /usr/src/app

COPY app /usr/src/app

RUN apt update
RUN DEBIAN_FRONTEND=noninteractive apt install -y protobuf-compiler python3-pil python3-lxml python3-pip
RUN pip3 install --upgrade pip
RUN pip3 install -r requirements.txt

CMD ./main.py
```

```
docker build . -t tensorflow-object-detection-example
docker tag tensorflow-object-detection-example markpengisme/tensorflow-object-detection-example:1.0.0
docker push markpengisme/tensorflow-object-detection-example:1.0.0
```

```
docker run -d -p 80:80 markpengisme/tensorflow-object-detection-example:1.0.0
```

Open <http://0.0.0.0> > Choose File > Upload
