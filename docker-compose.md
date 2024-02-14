# Example 1: (Volumes & Networks also will be created by docker compose)
### docker-compose.yml 
```docker-compose
version: '3.1'
services:
  app:
    image: mylandmarktech/myapp:6
    restart: always 
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
    driver: local
networks:
  wellsfargo:
    driver: bridge![image](https://github.com/itoroukpe/Dockerfiles/assets/122684000/b90b628b-0606-4f84-b3d4-987eb28b63bd)
```
