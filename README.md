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
### Run container with a name and in detached mode
```
docker container run --name something -d nginx
```
### Stop container
```
docker container stop name
```
### Stop all containers
```
docker container stop $(docker container ls -aq)
```
### Remove container
```
docker container rm nginx
```
If you want to run a container with the same name of a stopped container you need to remove that container first.




