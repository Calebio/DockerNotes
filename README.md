# DockerNotes

**VIDEO:** https://www.youtube.com/watch?v=3c-iBn73dDE&t=9814s </br>
NOTE: The demo in this video tutorial was done with older versions of mongodb image and mongo-express image
=> mongodb:4.2.1
=> mongo-express:0.49


Docker Notes
------------------------

- `docker ps` => list running containers
- `docker ps -a` => list of all containers running and stopped
- `docker run <image name>` => run image which will run as container in attached mode
- `docker run -d <image name>` => run container in detached mode and displays the container-id
- `docker run <image-name> --name <unique-name>` => to set your own unique name for the container
- `docker stop <container-id>` => stop container
- `docker start <container-id>` => start container
- `docker rmi <image-id>`  ==> to remove image
- `docker rm <container-id>` => to remove container.


Notes:
- When you restart a container, everything you have configured in the container/(app in the container) will be deleted/removed.


Container Port vs Host Port
---------------------------------------

- Multiple containers can run on your host machine
- Your laptop has only certain ports available
- Conflicts when you use the same port on host machine; for example, you can run multiple containers on port 3000(in this case 3 containers are running on port 3000), if you want to access these containers from your host machine you will have to bind each container to unique ports on the host machine i.e

OnHost(3001) => Oncontainer1(3000)
OnHost(3002) => Oncontainer1(3000)
OnHost(3003) => Oncontainer1(3000)


You can bind a host port to a container port when running the image:
- `docker run -p <host-port>:<container-port> <image-name>`

start in detached mode
- `docker run -p <host-port>:<container-port> -d <image-name>`


Q: 
How can I unbind a container after I bind it to a host port??



Debugging Containers
--------------------------------------------

- `docker logs <unique-name> Or <container-id>` ==> this is used to see logs in containers

- `docker logs <unique-name> Or <container-id> | tail` ==> to see the last part/event of the logs

- `docker logs <unique-name> Or <container-id> -f` ==> to stream logs and track changes as they are made

- `docker exec -it <container-id>OR<unique-name> <file/path>`

`docker exec` is used to access and navigate inside a container just to debug and fix issues in a container. 


Docker Network
------------------------------------

When docker containers are created, there's an isolated docker network that connects containers using the container name. This means when an application tries to access the containers in the docker isolated network, they will have to use localhost:<docker-port> e.g localhost:3000

- `docker network ls` => List docker network
- `docker network create <network-name>`  ==> To create a new docker network

*** Create Docker images on the same network ***

```
docker run -d \
-p 27017:27017 \
-e MONGO_INITDB_ROOT_USERNAME=admin \
-e MONGO_INITDB_ROOT_PASSWORD=password \
--name mongodb \
--net mongo-network \
mongo:4.2.1


docker run -d \
-p 8081:8081 \
-e ME_CONFIG_MONGODB_PORT=27017 \
-e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
-e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
--net mongo-network \
--name mongo-express \
-e ME_CONFIG_MONGODB_AUTH_DATABASE=admin \
-e ME_CONFIG_MONGODB_SERVER=mongodb \
mongo-express:0.49
```
NOTE: I specified/used these versions on mongodb and mongo-express because of compatibility issues



Docker Compose
--------------------------------------------
Is used to put docker commands together that would have been run one by one on the terminal.

- `docker-compose -f <docker-compose filename>.yaml up`  ==> to start containers using compose file
- `docker-compose -f <docker-compose filename>.yaml up -d`  ==> to containers in detached mode

- `docker-compose -f <docker-compose filename>.yaml down` ==> to stop containers


NOTE: docker-compose file creates a common network for the services so you don't have to create a docker network when using docker compose. Also removes the network once the compose is stopped

typical docker-compose syntax:

```
version: '<version-number>'
services:
  <service-name>:
    image: nginx:latest 
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html:ro  ##path persistent volume if you have one
    depends_on:
      - app ## specify as appropriate  
    networks:
      - my_network
    restart: always
  app:
    build: ./app
    environment:
      DATABASE_URL: postgres://user:password@db:5432/mydb
    volumes:
      - ./app:/app
    ports:
      - "3000:3000"
    networks:
      - my_network
    depends_on:
      - db
```



Dockerfile
--------------------------
Is a blueprint for building images mostly based on another image


typical Dockerfile syntax:

```
FROM <image-name>

ENV <specify env varialble> ## always advisable to have the env variables in the docker compose file so if there's a change you do it there and it will reflect in the containers.

RUN <Linux commands> typically => 'mkdir -p /home/app' and directory is created inside the container. This directory will typically have the files needed to run the application in the container.

COPY . /home/app  => copies file. This command runs on the host

# set default dir so that next commands executes in /home/app dir
WORKDIR /home/app


CMD ["node","server.js"] this serves as entry-point to the application, as the command the starts the app.

```

NOTE: 

RUN vs CMD
- RUN is used to execute commands in the container and you can have multiple RUNs
- CMD is used as entry-point into the application you can have just one.



Build Docker Image
------------------------------------

- `docker build -t <image-name>:version/tag <Dockerfile directory>`
e.g =>  `docker build -t my-custom-image:1.0 .`

When container is running custom image, and you want to check inside the file system of the custom container:

- `docker exec -it <container-id> /bin/sh`
e.g - `docker exec -it e91ef727e717 /bin/sh`


Pushing Custom Docker images to private repo(AWS/Azure)
-------------------------------------------------------------

- More information can be found in the respective docs of these cloud providers.


Docker Volumes
==================================================

- when do you need docker volumes?
=> when you need to persist data and be sure not to lose the data after restarting containers.

3 types of docker volumes
---------------------------------------------------

1. You Specify Host Volumes
with this method files are copied to the host to preserve it even after container restarts, you decide where on the host file system the reference is made.

this is done with docker run
syntax 
- `docker run... -v <host-path>:<container-path>`
e.g  `docker run.. -v /home/mount/data:/var/lib/mysql/data`


2. You don't Specify Host Volume
with the method, you only specify the container directory, you don't specify which directory on the host should be mounted. for each container a folder is generated that gets mounted(anonymous volumes)

this is done with docker run
syntax 
- `docker run... -v <container-path>`
e.g  `docker run.. -v /var/lib/mysql/data`




3. You can reference a volume by name

It is recommended that you use this in production.

this is done with docker run
syntax 
- `docker run... -v name:<container-path>`
e.g  `docker run.. -v name:/var/lib/mysql/data`


```
# Using docker-compose:

version: '3'
services :
mongodb:
image: mongo
ports:
- 27017:27017
volumes :
- db-data:/var/lib/mysqt/data
mongo-express :
image: mongo-express
volumes :
db-data
```

NOTE: It's important to note that the paths differs per database. Always check the default database path for your specific db online.

