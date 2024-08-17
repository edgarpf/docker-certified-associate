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
* To ensure a service runs on worker nodes rather than managers, use a node.role constraint in the service definition. For example: ```docker service create --name nginx-workers-only --constraint node.role==worker```.
* A number of tasks can be specified for a replicated service. There is no pre-specified number of tasks for global service.
* To remove a node from the swarm use the following on the node itself: ```docker swarm leave [OPTIONS]```.
* Sometimes, such as planned maintenance times, you need to set a node to DRAIN availability. DRAIN availability prevents a node from receiving new tasks from the swarm manager.
* By default, docker inspect will render results in a JSON array.
* By default, a container has no resource constraints and can use as much of a given resource as the host`s kernel scheduler allows.
* Swarm mode has two types of services: replicated and global. For replicated services, you specify the number of replica tasks for the swarm manager to schedule onto available nodes. For global services, the scheduler places one task on each available node that meets the service’s placement constraints and resource requirements.
* You can change almost everything about an existing service using the ```docker service update``` command. Use the --network-add or --network-rm flags to add or remove a network for a service.
* To update a service to add or update a mount on a service use: ```docker service update --mount-add SERVICE```.
* The scale command enables you to scale one or more replicated services either up or down to the desired number of replicas. This command cannot be applied to services which are global services.
* A task is the atomic unit of scheduling within a swarm. When you declare a desired service state by creating or updating a service via HTTP API endpoints, the orchestrator realizes the desired state by scheduling tasks.
* A stack is used to describe a collection of swarm services that are related, most probably because they are part of the same application.
* Us ```docker service update --constraint-add``` to add or update a placement constraint.
* To deploy your application to a swarm, you submit a service definition to a manager node. The manager node dispatches units of work called tasks to worker nodes.
* You can also specify the protocol to use, tcp, udp, or sctp. Defaults to tcp. To bind a port for both protocols, specify the -p or --publish flag twice. Ex: ```docker service create --name my_web --replicas 3 --publish 8080:80/udp nginx```.
* If the swarm loses the quorum of managers, the swarm cannot perform administrative tasks such as scaling or updating services and joining or removing nodes from the swarm.
* While placement constraints limit the nodes a service can run on, placement preferences try to place tasks on appropriate nodes in an algorithmic way (currently, only spread evenly).
* docker API > orchestrator > allocator > dispatcher > scheduler is the correct order of steps taken when creating a service process in swarm mode.
* Minikube is not a supported orchestrator in Docker EE.
* To add or update a node label (key=value) use: ```docker node update --label-add NODE```.
* Run ```docker node inspect self``` on the same node to confirm it has left the cluster.
* To configure the restart policy for a container, use the --restart flag when using the docker run command.
* Use --publish-add to publish containers port(s) on an existing service in the swarm.
* A task is a one-directional mechanism.
* Docker only allows a single network to be specified with the docker run command.
* Adding manager nodes to a swarm makes the swarm more fault-tolerant. However, additional manager nodes reduce write performance because more nodes must acknowledge proposals to update the swarm state. This means more network round-trip traffic.
* Setting a node to DRAIN does not remove standalone containers from that node, such as those created with docker run, docker-compose up, or the Docker Engine API.
* Restart policies only apply to containers. Restart policies for swarm services are configured differently using other flags related to service restart.
* Tasks are initialized in the NEW state.
* An odd number of managers is recommended.
* When using 'docker service create' and 'docker service update' you can set a --update-delayflag to delay between updates.
* To rollback a specified service to its previous version from the swarm you can execute one of the following commands: ```docker service rollback [OPTIONS] SERVICE``` or ```docker service update --rollback SERVICE```.
* Assuming that the my_web service exists, use the following command to remove the published port 80: ```docker service update --publish-rm 80 my_web```.
* Docker manager nodes store the swarm state and manager logs in the /var/lib/docker/swarm/ directory.
* Multi-stage builds allow you to drastically reduce the size of your final image, without struggling to reduce the number of intermediate layers and files.
* When building an image, do not use your root directory, /, as the PATH as it causes the build to transfer the entire contents of your hard drive to the Docker daemon.
* The build is run by the Docker daemon, not by the CLI.
* A Docker image consists of read-only layers each of which represents a Dockerfile instruction.
* ENTRYPOINT should be defined when using the container as an executable.
* You can use ```docker rmi```to remove an image.
* You can use an ARG or an ENV instruction to specify variables that are available to the RUN instruction.
* To show untagged images, or dangling, use: ```docker images --filter "dangling=true"```.
* Unlike an ARG instruction, ENV values are always persisted in the built image.
* Don’t confuse RUN with CMD. RUN actually runs a command and commits the result; CMD does not execute anything at build time, but specifies the intended command for the image.
* FROM can appear multiple times within a single Dockerfile to create multiple images or use one build stage as a dependency for another.
* By default, when the Docker daemon terminates, it shuts down running containers. Starting with Docker Engine 1.12, you can configure the daemon so that containers remain running if the daemon becomes unavailable.
* If there is more than one filter, then pass multiple flags (e.g. --filter is-automated=true --filter stars=3)
* To prevent tags from being overwritten, you can configure a repository to be immutable in the DTR web UI. Once configured, DTR will not allow anyone else to push another image tag with the same name.
* Layering RUN instructions and generating commits conforms to the core concepts of Docker where commits are cheap and containers can be created from any point in an image’s history, much like source control.
* By default, when the Docker daemon terminates, it shuts down running containers. Starting with Docker Engine 1.12, you can configure the daemon so that containers remain running if the daemon becomes unavailable.
* DTR also exposes several endpoints you can use to assess if a DTR replica is healthy or not:
  * /health: Checks if the several components of a DTR replica are healthy, and returns a simple json response. This is useful for load balancing or other automated health check tasks.
  * /load_balancer_status: Checks if the several components of a DTR replica can be reached, and displays that information in a table. This is useful for an administrator to gauge the status of a DTR replica.
  * /nginx_status: Returns the number of connections being handled by the NGINX front-end used by DTR.
  * /api/v0/meta/cluster_status: Returns extensive information about all DTR replicas.
* Docker Engine combines the namespaces, control groups, and UnionFS into a wrapper called a container format. The default container format is libcontainer.
* Images in DTR content are NOT backed-up.
* You can use the https://<ucp-manager-url>/_ping endpoint to check the health of a single UCP manager node.
* Use ```docker system events``` to get real-time events from the server.
* Docker Engine on Linux also relies on another technology called control groups (cgroups). A cgroup limits an application to a specific set of resources. Control groups allow Docker Engine to share available hardware resources to containers and optionally enforce limits and constraints.
* Linux namespaces provide isolation for running processes, limiting their access to system resources without the running process being aware of the limitations.
* Docker Engine uses namespaces such as the following on Linux:
  * The pid namespace: Process isolation (PID: Process ID).
  * The net namespace: Managing network interfaces (NET: Networking).
  * The ipc namespace: Managing access to IPC resources (IPC: InterProcess Communication).
  * The mnt namespace: Managing filesystem mount points (MNT: Mount).
  * The uts namespace: Isolating kernel and version identifiers. (UTS: Unix Timesharing System).
* Docker provides two modes for delivering messages from the container to the log driver:
  * (default) direct, blocking delivery from container to driver
  * non-blocking delivery that stores log messages in an intermediate per-container ring buffer for consumption by driver.
* If you want to use the self-signed certificate that UCP has out of the box, you can set up the SANs when you install UCP with the --san argument. You can also add them after installation.
* To display system-wide information use: ```docker info [OPTIONS]```.
* There are two ways to enable debugging. The recommended approach is to set the debug key to true in the daemon.json file. This method works for every Docker platform.
* You can install DTR on-premises or on a cloud provider. To install DTR, all nodes must:
  * Be a worker node managed by UCP (Universal Control Plane). DTR replicas are installed one on each UCP worker node.
  * Have a fixed hostname.
* There are a number of ways to configure the daemon flags and environment variables for your Docker daemon. The recommended way is to use the platform-independent daemon.json
* The main reason to use a DTR cache is so that users can pull images from a service that’s geographically closer to them.
* --default-ulimit allows you to set the default ulimit options to use for all containers. It takes the same options as --ulimit for docker run.
* --force-new-cluster: This flag forces an existing node that was part of a quorum that was lost to restart as a single node Manager without losing its data.
* If you don’t want to use a system utility to manage the Docker daemon, or just want to test things out, you can manually run it using the dockerd command.
* The default logging driver is json-file.
* Different types and use cases for the built-in network drivers:
  * User-defined bridge networks are best when you need multiple containers to communicate on the same Docker host.
  * Host networks are best when the network stack should not be isolated from the Docker host, but you want other aspects of the container to be isolated.
  * Overlay networks are best when you need containers running on different Docker hosts to communicate, or when multiple applications work together using swarm services.
  * Macvlan networks are best when you are migrating from a VM setup or need your containers to look like physical hosts on your network, each with a unique MAC address.
* By default, Docker Swarm uses a default address pool 10.0.0.0/8 for global scope (overlay) networks.
* To connect a running container to an existing user-defined bridge, use the docker network connect command.
* When you create a swarm service and do not connect it to a user-defined overlay network, it connects to the ingress network by default.
* Docker network drivers have a concept of scope. The network scope is the domain of the driver which can be the local or swarm scope.
* ClusterIP is the default method of exposing the service internally.
* Linked containers on the default bridge network share environment variables.
* Docker Universal Control Plane (UCP) uses Calico as the default Kubernetes networking solution.
* Ingress is one of the types of overlay networks.
* Services using the routing mesh are running in virtual IP (VIP) mode.
* UDP port 4789 is used on Docker host to allow overlay network traffic in swarm mode.
* If you use the host network mode for a container, that container’s network stack is not isolated from the Docker host (the container shares the host’s networking namespace), and the container does not get its own IP-address allocated.
* The overlay network driver creates a distributed network among multiple Docker daemon hosts.
* To specify a DNS server for an individual container use: ```docker container create --dns=IP_ADDRESS```. To set the DNS server for all Docker containers, use: ```dockerd --dns IP_ADDRESS```.
* Different types and use cases for the built-in network drivers:
  * User-defined bridge networks are best when you need multiple containers to communicate on the same Docker host.
  * Host networks are best when the network stack should not be isolated from the Docker host, but you want other aspects of the container to be isolated.
  * Overlay networks are best when you need containers running on different Docker hosts to communicate, or when multiple applications work together using swarm services.
  * Macvlan networks are best when you are migrating from a VM setup or need your containers to look like physical hosts on your network, each with a unique MAC address.
  * Third-party network plugins allow you to integrate Docker with specialized network stacks.
* The routing mesh enables each node in the swarm to accept connections on published ports for any service running in the swarm, even if there’s no task running on the node.
* In the event that a cluster CA key or a manager node is compromised, you can rotate the swarm root CA so that none of the nodes trust certificates signed by the old root CA anymore.
* The default roles in UCP are None, View Only, Restricted Control, Scheduler, and Full Control.
* When you deploy UCP, it starts running a globally scheduled service called ucp-agent.
* While it’s highly recommended to secure your registry using a TLS certificate issued by a known CA, you can choose to use self-signed certificates, or use your registry over an unencrypted HTTP connection. You can pass the --engine-insecure-registry flag when the docker engine is started.
* By default, the user inside the container created from this Dockerfile is root.
* Collections are groupings of objects within UCP. A collection can be made up of one or many of nodes, stacks, containers, services, volumes, networks, secrets, or configs — or it can hold other collections.
* Seccomp (short for Secure Computing Mode) is a security feature of the Linux kernel, used to restrict the syscalls available to a given process.
* After you create a secret, you cannot update it. You can only remove and re-create it, and you cannot remove a secret that a service is using.
* Secrets must be created on a swarm manager node.
* Docker UCP integrates with LDAP directory services, so that you can manage users and groups from your organization’s directory and it will automatically propagate that information to UCP and DTR.
* To use the Docker CLI with UCP, download a client certificate bundle(zip) by using the UCP web UI.
* When you initialize a new swarm, you can use the --autolock flag to enable autolocking of swarm manager nodes when Docker restarts. ```docker swarm init --autolock```.
* Docker Security Scanning is available as an add-on to Docker Trusted Registry, and an administrator configures it for your DTR instance.
* Linux capabilities are an even more granular way of reducing surface area. Docker Engine has a default list of capabilities that are kept for newly-created containers, and by using the --cap-drop option for docker run, users can exclude additional capabilities from being used by processes inside the container on a capability-by-capability basis. All privileges can be dropped with the --user option.
* Universal Control Plane's management plane uses a private CA and certificates for all internal communication. The client certificates are automatically rotated on a schedule, providing a strong method for reducing the effect of a compromised node. There is an option to reduce the default time interval of 90 days to a shorter interval, however shorter intervals do add stress to the UCP cluster.
* UCP issues different types of certificates depending on the user:
  * User certificate bundles: only allow running docker commands through a UCP manager node.
  * Admin user certificate bundles: allow running docker commands on the Docker Engine of any node.
* Docker Swarm comes with integrated PKI. All managers and nodes in the Swarm have a cryptographically signed identity in the form of a signed certificate. All manager-to-manager and manager-to-node control communication is secured out of the box with TLS.
* As of Docker Engine 18.06 there is a ```docker trust``` command that will streamline the image signing process.
* Ucp-agent service automatically starts serving all UCP components in manager node and only proxy service in worker node.
* If you prefer, you can specify your own externally-generated root CA, using the --external-ca flag of the docker swarm init command.
* 
