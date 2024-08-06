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

