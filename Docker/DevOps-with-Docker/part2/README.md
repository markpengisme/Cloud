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

