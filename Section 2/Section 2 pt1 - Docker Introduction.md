# Docker architecture

The architecture is as follows:

**Docker Client:**
- Docker Daemon:
  - Containers
  - Local images
  - Image registry

The Docker Client sends comands to the Docker Daemon. The Docker Deamon is the one actually responsible for carrying out those commands.

# Benefits of Docker

### Standardized Application Packaging

- all containers are packaged the same way and can be run the same way

### Multi Platform Support
- runnable on any platform that supports docker

### Light-Weight & Isolation
- large improvement on resources compared to virtual machines (no separate OS)
- containers are isolated; they can have individual settings and resource limits

> ⚠️ Good to know: Docker allows connecting individual containers together via a closed network

# Docker basics

Getting a list of pulled images:
```sh
docker images
```
- the list includes IDs of each image, making them referencable. 

Getting a list of containers (equivalent commands):
```sh
docker container ls
docker ps
```

1. docker container list
2. docker process status
   
- Show all containers (default shows just running):
    ```sh
    docker container ls -a
    docker ps -a
    ```

Running an image as a container:
```sh
docker run -p 5000:5000 in28min/hello-world-nodejs:0.0.1.RELEASE
```

>If the app inside a container uses a port different to 5000 (such as 8080 for REST API), make sure to switch the internal port in the run command!

 - With the ability to send exit signal with CTRL+C on windows:
    ```sh
    docker run -t -i ...
    ```

 - Detached (doesn't take control of the shell):
    ```sh
    docker run -d ...
    ```

## Useful container commands
For detached containers, or when referring to another shell's attached container.

>Commands working with container ID only require a prefix of the full ID  
>`(b5 is the same as the full ID b51131462758.....)`
> - This prefix must be unique from other containers

Show logs of container:
```sh
docker logs <container ID>
```
 - Actively follow these logs:
    ```sh
    docker logs -f ...
    ```
     This way, CTRL+C detaches the logs instead of stopping the container.

Stop container:
```sh
docker container stop <container ID>
```

# Docker Images (In-Depth)

Commands for managing and working with images.

Pull image:
```sh
docker pull <path to image>
```
- downloads :latest if version tag is not provided


Search images from docker hub:
```sh
docker search <>
```
- also shows if an image is official - more reliable 


Show the details of inner layers of an image:
```sh
docker image history <image>
```

Show details of an image:
```sh
docker image inspect <image>
```

Remove a locally pulled image:
```sh
docker image remove <image>
```
- not allowed if there is a container that is referencing the image, removing the image requires the following (even if the container is shut down):
  ```sh
  docker container rm <ID of container>
  ```
- every single disabled container that ever referenced this image needs to be removed this way first

# Docker Containers (In-Depth)

The command
```sh
docker run ...
```
is actually a shortening of:

```sh
docker container run ...
```


Pause a container:
```sh
docker container pause <container ID>
```
- like stop, but only temporarily disables the container, with the ability to unpause:
    ```
    docker container unpause <container ID>
    ```

Immediately kill a container:
```sh
docker container kill <container ID>
```
- **"docker container stop"** lets the container shut down gracefully, such as the container asking if you want to save your work before shutdown (SIGTERM)
- **"docker container kill"** immediately kills the container (SIGKILL)


Show details of a container:
```sh
docker container inspect <container ID>
```
- shows the ports, details of the bridge network, among many other attributes of the container
- usable for troubleshooting a container
  

Remove all stopped containers:
```sh
docker container prune
```

# Docker System

Show Docker's disk usage:

```sh
docker system df
```
 - Output:

    ```
    TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
    Images          3         0         311.5MB   311.5MB (100%)
    Containers      0         0         0B        0B
    Local Volumes   3         0         0B        0B
    Build Cache     0         0         0B        0B
    ```


Show Docker events happening in real time:

```sh
docker system events
```
- see events when an image is pulled, a container is started and killed, etc.


Prune all the stopped containers, images with no containers attached, unused networks, and build cache (all currently unused resources):

```sh
docker system prune -a
```
- usable for freeing up space quickly


# Managing memory and CPU usage

Find usage stats for a container, to see if we need to modify its resources:

```sh
docker stats <container>
```
- Output:
  
  ```
  CONTAINER ID   NAME                 CPU %     MEM USAGE / LIMIT     MEM %     NET I/O     BLOCK I/O   PIDS
  85d604d62cff   affectionate_euler   0.00%     15.77MiB / 6.705GiB   0.23%     876B / 0B   0B / 0B     6
  ```
- the container is allowed to use up to 6.70 GiB of memory!

To modify the memory dedicated to a container at startup:

```
docker run -p 5000:5000 -d -m 512m ...
```

To modify the CPU usage dedicated to a container at startup:

```
docker run -p 5000:5000 -d --cpu-quota=50000 ...
```
- the total CPU quota is defined as 100,000, so we are allowing 50% usage at maximum
- the downside is that the startup of the container is slower this way, as by default it temporarily uses more than 50%

# Troubleshooting

**Exception: java.net.ConnectException: Connection refused: connect**

If you are using Windows 10 and docker version 2.0.0.3 (31259) or above, you need to enable Expose Daemon without TLS option.

- Step 1: Right click on the Docker Desktop icon on the right of your task bar

- Step 2: Click on Settings

- Step 3: In the General tab, click checkbox “Expose Daemon on tcp://localhost:2375 without TLS”
