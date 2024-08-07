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





