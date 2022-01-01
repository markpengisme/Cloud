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

# 2.3

```yaml
version: '3'  

services: 
    frontend:
        image: example-frontend
        ports: 
            - 5000:5000
    backend: 
        image: example-backend
        ports: 
            - 8080:8080
```

http://localhost:5000

# 2.4

![img](https://docker-hy.github.io/images/exercises/back-front-and-redis.png)

```yaml
version: '3'  

services: 
    frontend:
        image: example-frontend
        ports: 
            - 5000:5000
    backend: 
        image: example-backend
        environment:
            - REDIS_HOST=cache
            - REDIS_PORT=6379
        ports: 
            - 8080:8080
    cache:
        image: redis
```

http://localhost:5000
