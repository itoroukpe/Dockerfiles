# Example 2: (Volumes & Networks will not be created by docker compose.As we set volumes and networks as external)
```docker-compose
version: '3.1'
services:
  app:
    #replicas: 4
    image: rondustech/myapp:6
    restart: always # This will be ignored if we deploy in docker swarm
    environment:
    - MONGO_DB_HOSTNAME=mongod
    - MONGO_DB_USERNAME=devdb
    - MONGO_DB_PASSWORD=devdb123
    ports:
      - 8080:8080
    networks:
    - wellsfargo
  mongod:
    image: mongo
    environment:
    - MONGO_INITDB_ROOT_USERNAME=devdb
    - MONGO_INITDB_ROOT_PASSWORD=devdb123
    volumes:
      - mydatas:/data/db
    restart: always
    networks:
    - wellsfargo
volumes:
  mydatas:
    external: true
networks:
  wellsfargo:
    external: true![image](https://github.com/itoroukpe/Dockerfiles/assets/122684000/ace72b7e-cfa7-42b8-b58d-0031344e9afd)
```
