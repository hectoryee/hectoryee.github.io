---
title: Running Docker
excerpt: Run jupyter with Tensorflow inside Docker.
published: true
date: 2019-03-02
categories: project
tags: jupyter tensorflow docker
---

## My Running Version
``` bash
docker run --runtime=nvidia -it -w /notebooks -p 8888:8888 -v $(realpath ~/project/):/notebooks -e HOST_PERMS="$(id -u):$(id -g)"     tensorflow/tensorflow:nightly-devel-gpu-py3 bash

docker start -ai <container_name>

jupyter notebook --port=8888 --ip=0.0.0.0 --allow-root --no-browser .
```

## [Using a Jupyter Notebook within a Docker Container](https://devtalk.nvidia.com/default/topic/1032202/docker-and-nvidia-docker/using-a-jupyter-notebook-within-a-docker-container/)
``` bash
docker run --runtime=nvidia -it -v "/my-local-computer-files/:/my-docker-container/" my-nvidia-container
```

Add flag
- `-p 8888:8888`
- `-v "/my-local-computer-files/:/my-docker-container/"`

Example
``` bash
docker run --runtime=nvidia -it -p "8888:8888" -v "/my-local-computer-files/:/my-docker-container/" my-nvidia-container
```

Inside docker run:
``` bash
jupyter notebook --port=8888 --ip=0.0.0.0 --allow-root --no-browser .
```

## [Running Docker](https://stackify.com/docker-tutorial/)
``` bash
# Run a Docker image
docker run hello-world

# List images
docker image ls

# List the containers on our system
docker ps -a

# List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq

# Reuse a container
docker start -attach <container name>

# Look inside the container and show the running processes
docker top <container name>

# Stop container
docker stop <container name>

# Remove docker
docker rm <container name>

```
- `-attach` tells Docker to connect to the container output so we can see the results
- `-it` flags allow us to interact with the shell.

### Share system resources with a container
``` bash
docker run -v /full/path/to/html/directory:/usr/share/nginx/html:ro -p 8080:80 -d nginx
```
- `-v /full/path/to/html/directory:/usr/share/nginx/html:ro` maps the directory holding our web page to the required location in the image. The ro field instructs Docker to mount it in read-only mode. It’s best to pass Docker the full paths when specifying host directories.
- `-p 8080:80` maps network service port 80 in the container to 8080 on our host system.
- `-d` detaches the container from our command line session. Unlike our previous two examples, we don’t want to interact with this container.
- `nginx` is the name of the image.

## [Cleaning Docker containers](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes)

### Purging All Unused or Dangling Images, Containers, Volumes, and Networks
``` bash
docker system prune
```

To additionally remove any stopped containers and all unused images (not just dangling images), add the `-a` flag to the command:
``` bash
docker system prune -a
```

### Removing Docker Images
#### Remove one or more specific images
List:
``` bash
docker images -a
```

Remove:
``` bash
docker rmi Image Image
```

#### Remove dangling images
Docker images consist of multiple layers. Dangling images are layers that have no relationship to any tagged images.

List:
``` bash
docker images -f dangling=true
```

Remove:
``` bash
docker images purge
```

#### Remove all images
List:
``` bash
docker images -a
```

Remove:
``` bash
docker rmi $(docker images -a -q)
```

### Removing Containers
#### Remove one or more specific containers
List:
``` bash
docker ps -a
```

Remove:
``` bash
docker rm ID_or_Name ID_or_Name
```

#### Stop and remove all containers
List:
``` bash
docker ps -a
```

Remove:
``` bash
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
```

## Common Issues
- [How to fix docker: Got permission denied issue](https://stackoverflow.com/questions/48957195/how-to-fix-docker-got-permission-denied-issue)
    - Create the docker group.
    ```sudo groupadd docker```
    - Add your user to the docker group.
    ```sudo usermod -aG docker $USER```
    - Logout and login again and run.
    ```docker run hello-world```

