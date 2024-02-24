# Docker Volumes

### Volumes:
=======
= java-based 
= spring-boot
= container run time docker


1) Create docker network using below commond(If it's not created already)

     docker network create  -d bridge ebay-network

2) Create a mongo contianer without volume in above network
	 	 
	 docker run -d --name mongo -e MONGO_INITDB_ROOT_USERNAME=devdb -e MONGO_INITDB_ROOT_PASSWORD=devdb1234  --network ebay-network mongo

3) Create Spring Application Container in above network & which will talk to mongo data base container

       docker run -d -p 8081:8080 --name springapp  -e MONGO_DB_HOSTNAME=mongo -e MONGO_DB_USERNAME=devdb -e MONGO_DB_PASSWORD=devdb1234 --network ebay-network mylandmarktech/myapp:6
       
    
	 
4) Access Spring application & insert data it will be inserted to mongodb. Delete and recreate mongo container 
   what ever you have inserted will no longer be avaiable. As once we delete contaienr data also will be deleted in container.
   
   To take data backup from container we have to use volunmes
   
Bind Mounts:
Bind mounts may be stored anywhere on the host system. They may even be important system files or directories. Non-Docker processes on the Docker host or a Docker container can modify them at any time.

### Volumes Using bind mount
```
mkdir data &&
docker run -d --name mongo -v data:/data/db  -e MONGO_INITDB_ROOT_USERNAME=devdb -e MONGO_INITDB_ROOT_PASSWORD=devdb1234  --network ebay-network mongo
```

Docker Volumes
Volumes are stored in a part of the host filesystem which is managed by Docker service (/var/lib/docker/volumes/ on Linux).
Non-Docker processes should not modify this part of the filesystem. Volumes are the best way to persist data in Docker.
5) Create a volume a Local Volume(Execute docker volume ls to check existing volumes)
```
     docker volume create DBbackup
```

	### To list volumes
  
  ```
	 docker volume ls
   ```
6) Use above volume while creating container. Remove containers previously created
=========================================
 Create a mongo contianer without volume in above network
```
	 	 
	 docker run -d --name mongo -v DBbackup:/data/db   -e MONGO_INITDB_ROOT_USERNAME=devdb -e MONGO_INITDB_ROOT_PASSWORD=devdb1234  --network ebay-network mongo
```
 Create Spring Application Container in above network & which will talk to mongo data base container
```
       docker run -d -p 8081:8080 --name springapp  -e MONGO_DB_HOSTNAME=mongo -e MONGO_DB_USERNAME=devdb -e MONGO_DB_PASSWORD=devdb1234 --network ebay-network mylandmarktech/myapp:6
```       
  	 
   
7) Access Spring application & insert data it will be inserted to mongodb. Delete and recreate mongo container with the same volume mapping and you can retrieve your data.
   
   
    
 ===== Network Volumes Using AWS EBS==========
 Note: No plugin found [SKIP]
 
 1) Create IAM User with EC2 Full Access and user access key & Secret Key of the same. Replace your access key & secret below.

 docker plugin install rexray/ebs EBS_ACCESSKEY=$ACCESSKEY EBS_SECRETKEY=$SECRETKEY
 
 docker volume create --driver rexray/ebs --name ebsvolume
 
 docker volume create -d rexray/ebs ebsvolume

 docker run -d -p 27017:27017 -v ebsvolume:/data/db  --name mongo -e MONGO_INITDB_ROOT_USERNAME=devdb -e MONGO_INITDB_ROOT_PASSWORD=devdb1234  --network ebs-network mongo

By default volumes are mounted as Read/Write
We can map Volumes As Read Only using below option

   -v <volumeName/BindMount>:<containerPath>:ro
 ```  
   docker run -d -p 27017:27017 -v ebsvolume:/data/db:ro  --name mongo -e MONGO_INITDB_ROOT_USERNAME=devdb -e MONGO_INITDB_ROOT_PASSWORD=devdb1234  --network ebs-network mongo
```
 ======================================  
  
  
Docker Compose
==============

Docker Compose is a tool for defining and running multiple containerized applications.
In real time one application can have more than 7 micro-services:
ebay.com: 
An e-commerce java based web application for ebay or 
    Login
    Registration
    checkout
    Payment
    Order

An online banking java based web application for BOA or

With docker compose to deploy above applications which has only 2 images we executed below commnads.

```
docker network create -d bridge springappnetwork

docker volume create -d local mongobkp

 docker run -d --name mongo -v mongobkp:/data/db -e MONGO_INITDB_ROOT_USERNAME=devdb -e MONGO_INITDB_ROOT_PASSWORD=devdb1234  --network boa-network mongo
 
 docker run -d -p 8080:8080 --name springapp  -e MONGO_DB_HOSTNAME=mongo -e MONGO_DB_USERNAME=devdb -e MONGO_DB_PASSWORD=devdb1234 --network boa-network mylandmarktech/spring-boot-mongo
``` 

With Docker Compose we deploy/create all of the above 4 with single command using compose file.

==================================
With Compose

Install docker compose using below command:

sudo apt install docker-compose

We will define all the serivces(containers) details in compose file using compose file we can deploy multi container applications.

Defautl name : docker-compose.yml or docker-compose.yaml


Example 1: (Volumes & Networks also will be created by docker compose)
```
# docker-compose.yml 

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
    driver: bridge
```

Commands
# Syntax Check
docker-compose config 
# Create Services/Contianers
docker-compose up -d  
  
# Remove Services/Contianers 
docker-compose down	  

Example 2: (Volumes & Networks will not be created by docker compose.As we set volumes and networks as external)
==========
```
version: '3.1'

services:
  app:
    #replicas: 4
    image: mylandmarktech/myapp:6
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
    external: true
```	
If docker compose file with custom name.	
docker-compose -f <CustomeComposeFileName>.yml <command>

Ex:
docker-compose -f docker-compose-springapp.yml config
docker-compose -f docker-compose-springapp.yml up -d

docker-compose -f docker-compose-springapp.yml down


Docker Compose Commands:

  config             Validate and view the Compose file
  create             Create services
  down               Stop and remove containers, networks, images, and volumes
  exec               Execute a command in a running container
  help               Get help on a command
  images             List images
  kill               Kill containers
  logs               View output from containers
  pause              Pause services
  port               Print the public port for a port binding
  ps                 List containers
  pull               Pull service images
  push               Push service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  top                Display the running processes
  unpause            Unpause services
  up                 Create and start containers
  version            Show the Docker-Compose version information




# In Normal(Standalone) Docker Server We can use below command to create a containers.
docker-compose up

# In docker swarm we will use below command to deploy services using docker compose.
docker stack deploy --compose-file docker-compose.yml <stackName>

How do you manage deployments in your projects?  :
 
   Virtual Manchines VM   == EC2 
      Tomcat or JBoss application servers 
   Containerisation  = Docker
                     = Docker-compose
                     
   Container orchestra / Manager  = Docker Swarm
	
Docker Swarm:
=============


Containerization Tools: docker,rocker(rkt),coreos

Container Orchestration Tools: docker swarm,kubernetes,openshift ..etc


H.A --> High Availability
F.T --> Fault Tolarence
Scalability
L.B



# In Normal(Standalone) Docker Server We can use below command to create a container.
docker run -d -p <hostPort>:<containerPort> --name <containerName> <imageName>

# In docker swarm we will use below command to deploy service.
docker service create --name <serviceName> -p <hostPort>:<containerPort>  <imageName>

# In Normal(Standalone) Docker Server We can use below command to create a containers from compose.
docker-compose up

# In docker swarm we will use below command to deploy services using docker compose.
docker stack deploy --compose-file docker-compose.yml <stackName>


Note: Make Sure You Open Required/All Ports in AWS Security Groups. 8:2

======================================================================
# Initialize docker swarm cluster by exeuting below command on docker server which you want make it as Manager

# Initialize docker swarm on privateIP
docker swarm init

# Initialyze Docker swarm with Public IP
Note: Don't use below(If restart your systems public ip will change will break your cluster) use above commond to initilaze cluster.

# Intilaize docker swarm cluster on public IP
docker swarm init  --listen-addr=eth0 --advertise-addr $(curl http://169.254.169.254/latest/meta-data/public-ipv4)

=============================
Rerequisites:
============

3 - Ubuntu Servers
     1 - Master
     2 - Workers
   
Steps:
======

MASTER:

docker swarm init 
  
WORKER NODES:
#!/bin/bash
sudo hostname node1
sudo apt update -y
sudo apt install docker.io -y
sudo usermod -aG docker ubuntu

docker swarm join --token SWMTKN-1-36cfenl1orybxszlb2jvob063w6crqs8wojatnzz8ygxi10a87-6s46n61rw6cd4bpuii10z082w 172.31.13.25:2377

# Get worker token
docker swarm join-token worker (Get the token in manager & exeute in nodes)

docker swarm init
1) Install Docker Community Edition in all the 3 Servers.
   And add user(ubuntu) to docker group.
                
#!/bin/bash
sudo apt update -y
sudo apt install docker.io -y
# Add Current User to docker group.
sudo usermod -aG docker ubuntu

 # docker swarm join --token SWMTKN-1-0zjjon5c0d79sklkwkccp9klj8n1vdrxtv60b6s8mhd3r3d9qt-7pt3lsjdr0q6530yn0x4udghm 172.31.77.29:2377
       

2) Open below ports in AWS Security Groups.
      
    TCP port 2377 for cluster management communications
    TCP and UDP port 7946 for communication among nodes
    UDP port 4789 for overlay network traffic.
   
    If you are planning on creating an overlay network with encryption     (--opt encrypted), you will  also need     to ensure ip protocol 50 (ESP) traffic is allowed.
     
3) Execute below command to initiate docker swarm manager in one system.
  
   # Initialize docker swarm cluster.
   docker swarm init
  
   # Get worker token
   docker swarm join-token worker
  
   # Get manager token to add another node as secondary manager
   docker swarm join-token manager
  
4) Add workers machines to cluster.
   #Execute join token(worker token from manager) in all worker machines.
  
  
5) Display docker cluster nodes in manager machine.

   docker node ls
  
6) Deploy Sample Docker Application in docker swarm cluster.

    docker service create --name webserver --replicas 2 -p 80:80 httpd     

Docker Swarm has two modes

Global   --> All the nodes (3 servers 1 Manager + 2 Workers)
             use for monitoring softwares deployments 
               New Relic / Prometheus  / Cloudwatch
Replicas --> It will deploy based on replicated number.



What is service in docker or docker swarm?

Serivce is nothing but a collection of one or more replicas(contianers) of same type(Image).


What is stack in docker or docker swarm?

Stack is nothing but a collection of one or more serivces.

Docker Service Create
==================

docker service create  -p 8080:8080 --name javawebapp --replicas 2 mylandmarktech/java-web-app

Constraints
===========

# User constriants to create containers in specific docker hosts based on condition

docker service create  -p 8080:8080 --name javawebapp --replicas 2 --constraint 'node.role==worker' mylandmarktech/java-web-app

Rolling Update
==============
# Create a service with a rolling update policy
 deployment = 40
     version 1  -= 25%
     version 2  += 25%
       
docker service create -p 8080:8080 --replicas 4  --name javawebapp  --update-delay 30s --update-parallelism 1  mylandmarktech/java-web-app

Update
=======

docker service update --image mylandmarktech/myapp:6 javawebapp

docker service update --image mylandmarktech/java-web-app:2 javawebapp

with Volume
===========
# Create a service with Volume mapping
docker service create  -p <hostPort>:<containerPort> --name <serviceName> --replicas 1  --mount type=volume,source=<volumeName>,destination=<containerfolderPath> <imageName>

# List Services
=================
docker service ls

# List Services process
=======================
docker service ps <servicenName>

# Scale Services 
================
docker service scale javawebapp=3

#Remove Services
================
docker service rm javawebapp


# Add labels to node
docker node update --label-add key=value <nodeid>
Ex: docker node update --label-add server=nodeone qmdh9tgvdef99sryhbezswfl9

docker node update --label-add server=nodetwo  n2brbkq0h61a1m1i0wz8dsbbp

#Use above label in constrainsts
docker service create  -p 8000:8080 --name myapp --replicas 2 --constraint 'node.labels.server==nodetwo' mylandmarktech/maven-web-app

# Global mode 
docker service create  -p 9000:8080 --name myapp2 --mode=global   mylandmarktech/maven-web-app

# Drain Nodes in Cluster(Swarm will not create containers in drained nodes)

docker node update --availability drain <NodeID>


docker stack deploy --compose-file docker-compose.yml springmongo

docker stack ls

docker stack rm <stackName>


version: '3.1'

services:
  springboot:
    image: mylandmarktech/myapp:6
    restart: always # This will be ignored if we deploy in docker swarm
    container_name: springboot
    environment:
    - MONGO_DB_HOSTNAME=mongo
    - MONGO_DB_USERNAME=devdb
    - MONGO_DB_PASSWORD=devdb1234
    ports:
      - 8080:8080
    working_dir: /opt/app
    depends_on:
    - mongo
    deploy:  # This will be considered only in docker swarm.
      replicas: 2
      update_config:
        parallelism: 1
        delay: 20s
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 5
    networks:
    - wellsfargo
  mongo:
    image: mongo
    container_name: springboot-mongo
    environment:
    - MONGO_INITDB_ROOT_USERNAME=devdb
    - MONGO_INITDB_ROOT_PASSWORD=devdb1234
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
    external: true


# To come out of swarm execute below commond in worker node
docker swarm leave

# Remove node from Manager
docker node rm <nodename>

# Use private repo's.
#Send registry authentication details to swarm agents using --with-registry-auth	

If you are using insecure(http) registry. Make sure you have follow the below step in all servers (jenkins,docker swarm master,workers). Below Steps from 1 to 6 are not requried in real time since we will have secure(https) repositories.

1.	Login as root user
2.	Go to /etc/docker
      cd /etc/docker
3.	Then create a file called daemon.json 
      vi /etc/docker/daemon.json
4.	Write these script in daemon.json

{
  "insecure-registries": [ "<IPOfPrivateRepo>:<dockerRepoPort>" ]
}

ex:

{
  "insecure-registries": [ "172.31.45.81:8083" ]
}

Note: Replace with your nexus ip instead of 172.31.45.81. Make Sure You Opened 8083 port in nexus server security
group.

(Here we are allowing our docker daemon to access the Nexus Hosted Repo)

5.	Save the file 

6.	Restart docker
sudo systemctl restart docker


Before Pushing Image From Jenins execute docker login

docker login -u <username> -p <password>  <URL>
Ex:
docker login -u admin -p admin123 172.31.45.81:8083	
	
Execute docker login only in master

docker login -u <username> -p <password>  <URL>

docker login -u admin -p admin123 172.31.45.81:8083	

# While Creating a Service Or Stack send registry authentication details to swarm agents using --with-registry-auth	
	
docker service create  -p <hostPort>:<containerPort> --name <serviceName> --replicas 1  --with-registry-auth <imageName>

docker stack deploy --compose-file docker-compose.yml  --with-registry-auth <stackName>
