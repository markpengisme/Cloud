 # 2.1 

```yaml
version: '3'

services:
  simple-web-service:
    image: devopsdockeruh/simple-web-service
    volumes:
      - ./logs.txt:/usr/src/app/text.log
    container_name: simple-web-service
```

```
touch logs
docker-compose run simple-web-service
```

# 2.2

```yaml
  version: "3.0"
  
  services:
      web:
          image: devopsdockeruh/simple-web-service
          ports: 
              - 80:8080 
          command: server
          container_name: web-server
```

  ```sh
  docker-compose up
  ```

  http://0.0.0.0
