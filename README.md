# docker-certified-associate

### Show running containers
```
docker ps
docker container ls
```
### Also show stopped containers
```
docker ps -a
```
### List images
```
docker images
docker image ls
```
### Run container with a name, in detached mode and delete contaner after exit
```
docker container run -rm --name something -d nginx
```
### Stop container
```
docker container stop name
```
### Start a stopped container
```
docker container start name
```
### Stop all containers
```
docker stop $(docker container ls -aq)
docker container stop $(docker container ls -aq)
```
### Remove container
```
docker container rm nginx
```
If you want to run a container with the same name of a stopped container you need to remove that container first.

### Exec in container
```
docker container exec -it container_name bash
docker container exec -it container_name sh
```
### Override default container commands
```
docker container run -d nginx sleep 20
```
### [Restart policies](https://docs.docker.com/config/containers/start-containers-automatically/)
```
docker container run -d nginx --restart unless-stopped
```
### Disk usage metrics
```
docker system df
```
### Logs
```
docker logs container_name
```
### Dockerfile
```dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install -y nginx
COPY index.html /var/www/html
CMD nginx -g 'daemon off;'
```
COPY takes in a src and destination. It only lets you copy in a local file or directory from your host.

ADD lets you do that too, but it also supports 2 other sources:
* First, you can use a URL instead of a local file / directory.
* Secondly, you can extract a tar file from the source directly into the destination.

Using ADD to fetch packages from remote URLs is strongly discouraged; you should use curl or wget instead.

The EXPOSE instruction does not actually publish the port. It functions as a type of documentation

#### Healthcheck
```dockerfile
#Default values
#--interval=30s --timeout-30s --retries=3 -start-period=0
HEALTHCHECK --interval=5m --timeout=3s CMD curl -f http://localhost/ || exit 1
```
The best use for ENTRYPOINT is to set the image’s main command ENTRYPOINT doesn’t allow you to override the command.

It is important to understand distinction between CMD and ENTRYPOINT.

The WORKDIR instruction sets the working directory for any RUN, CMD, ENTRYPOINT, COPY and ADD instructions that follow it in the Dockerfile.
The WORKDIR instruction can be used multiple times in a Dockerfile.

#### ENV
```dockerfile
FROM busybox
ENV NGINX 1.2
RUN touch web-$NGINX.txt
CMD ["/bin/sh"]
```
```
#-e or --env-file
docker run -dt --name env02 --env USER=ADMINUSER busybox sh
```
#### TAG
```
docker build . -t demo:v1
```
#### Commit
```
docker container commit old_name new_name
docker container commit --change 'something' old_name
```
#### Pull
```
docker image pull image_name
docker pull image_name 
```
#### Pruning
```
docker image prune 
```
The command will remove only images which does not have tags or running container associated.
#### Flatenning
```
docker image history image_name
docker export image_name > image_name.tar
docker save image_name > image_name.tar
docker load < image_name.tar
cat image_name.tar | docker import - image_name:latest
```
#### Push
```
docker login
docker tag image_name something:tag
docker push tag_name
```
#### Search
```
docker search nginx --limit 5 --filter "ïs-official=true"
```
#### Network
```
docker network ls
docker inspect network_name
docker network create -d overlay -o encrypted mynetwork
docker network create --driver bridge network_name #bridge is the default
docker container run -dt image_name --network network_name --name some_name
docker container run -dt image_name --name some_name -P (random ports)

docker container run -dt image_name --link some_name02:container --name some_name (lecy approach)
```
#### Container orchestration
```
docker service ls
docker service ps service_name
```
### Docker Swarm
* To deploy your application to a swarm, you submit a service definition to a manager node.
* The manager node dispatches units of work called tasks to worker nodes.
```
docker swarm init --advertise-addr <MANAGER-IP>
docker swarm join-token worker
docker node ls
docker service create --name webserver --replicas 1 nginx
docker service ls
docker service rm service_name
docker service ps service_name
docker service scale service01=3 service02=3 # or docker service update --replicas 5 mywebsever
docker node update --availability drain swarm03
docker node update --availability active swarm03
docker service inspect demotroubleshoot --pretty
docker node inspect swarm01 -pretty
docker service create --name mywebserver --replicas 1 --publish 8080:80 nginx
```
There are two types of services deployments, replicated and global. A global service is a service that runs one task on every node.
```
docker service create --name webserver -dt nginx --mode global
```
### Docker Compose
```
docker-compose version
docker-compose up -d
docker-compose down
```
```yaml
version: '3.3'
 
services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
 
   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
       WORDPRESS_DB_NAME: wordpress
volumes:
    db_data: {}
```
### Docker Stack
The docker stack can be used to manage a multi-service application.
```
docker stack deploy --compose-file docker-compose.yml mydemo
docker stack ls
docker stack rm mydemo
```
### Autolock
```
docker swarm update --autolock=true
systemctl restart docker
docker swarm unlock
docker swarm unlock-key
docker swarm unlock-key --rotate
```
### Volumes
```
docker service create --name myservice --mount type=volume,source=myvolume,target=/mypath nginx
docker volume ls
```
### Security
```
docker swarm ca --rotate
docker container run -dt --name constraint01 --cpus=1.5 busybox sh
docker container run -dt --name constraint02 --cpuset-cpus=0,1 busybox sh
```
### Volumes
```
docker volume ls
docker volume create myvolume
docker container run -dt --name busybox -v myvolume:/etc busybox sh
docker volume rm myvolume
#bind olumes
docker container run -dt --name mynginx --mount type=bind,source=/root/index,target=/usr/share/nginx/html nginx
docker container run -dt --name mynginx -v /root/index:/usr/share/nginx/html nginx
docker container run -td --rm --name container busybox ping -c10 www.google.com #remove container and volume after completion
```
A given volume can be mounted into multiple containers simultaneously.
#### Logs
```
docker logs container_name
```

## Tips
* DOCKER CONTENT TRUST means that only trust image which are signed by a trust signer.
* DTR does backup images within the repository.
* user namespace is not enabled by default.
* Use ```docker system df``` to check disk occuppied for containers.
* Use ```docker swarm leave``` to remove a server from the group.
* To make sure that an image tag is not overwritten in Docker DTR use immutable option in DTR.
* To update a secret create a new docker secret with a new password. Trigger a rolling update of the service, by using "-- secret-rm" & "--secret-add" to remove the old secret and add the updated secret.
* A running container can be interpreted as a process on a Linux host.
* Universal Control Planne (UCP) is tool to use to create users and teams.
* ADD can copy files from local source to destination. It can also uncompress files. It can also copy files from remote sources.
* ```docker service update --replicas=3 web``` is the same as ```docker service scale web=3```.
* DOCKER_CONTENT_TRUST=1 is used to instruct the client to perform signing of the image.
* Garbage collection removes unreferenced image layers from DTR's backend storage.
* ```docker swarm init``` to setup a swarm.
* ```docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]```to tag an image.
* By default, ports are published using ingress mode. This means that the swarm routing mesh makes the service accessible at the published port on every node regardless if there is a task for the service running on the node.
* ```-memory 4GB --memory-reservation 2GB``` for a service would allow a container to consume more than 2 GB of memory only when there is no memory contention but would also prevent a container from consuming more than 4GB of memory.
* 

