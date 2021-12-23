# TensorFlow Object Detection APP

This project is a containerization of the [tensorflow-object-detection-example](`https://github.com/GoogleCloudPlatform/tensorflow-object-detection-example`).

## How to use

- Build by file , or pull from dockerhub.

```sh
docker build . -t markpengisme/tensorflow-object-detection-example:1.0.0
## or
docker pull markpengisme/tensorflow-object-detection-example:1.0.0
```

- Run

```sh
docker run -p 80:80 markpengisme/tensorflow-object-detection-example:1.0.0
```

- Open <http://0.0.0.0> > Choose File > Upload
