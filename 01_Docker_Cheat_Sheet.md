# Docker Cheat-Sheet

This cheat-sheet is "highly" using references from the official Docker documentation and the very great introductory [video](https://www.youtube.com/watch?v=zJ6WbK9zFpI) made by [Mumshad Mannambeth](https://github.com/mmumshad).

### This cheat-sheet contains the following sections :

- [Installing Docker on Ubuntu 18.04](#installing-docker-on-ubuntu-1804)
- [Basic Commands](#basic-commands)
- [A Bit Less Basic Commands](#a-bit-less-basic-commands)
- [Environment Variables](#environment-variables)
- [Creating a Docker Image](#creating-a-docker-image)
- [CMD vs ENTRYPOINT](#cmd-vs-entrypoint)
- [Networking](#networking)
- [Docker Storage and File System](#docker-storage-and-file-system)
- [Docker Compose](docker-compose)

## Installing Docker on Ubuntu 18.04

```bash
# Reference : https://docs.docker.com/v17.12/install/linux/docker-ce/ubuntu/#install-using-the-repository
# Remove
sudo apt-get remove docker docker-engine docker.io

# Setup repo

#update package index
sudo apt-get update

# Allows apt using https
sudo apt install \
	apt-transport-https \ 
	ca-certificates \
	curl \
	software-properties-common

# Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Setup a stable repo for docker
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"

# update package
sudo apt-get update

# Install docker
sudo apt-get install docker-ce
```

In case you're using Mac or Windows, you have just to install the Docker Desktop application. That's been said, it is better to use Docker on a Linux machine (Docker Desktop is running on top of a Linux virtualization).



## Basic Commands

```bash
# get docker version
docker version

# show all containers
docker ps

# showing even the ones not running
docker ps -a
docker container ls

# show all existing images 
docker images
docker image ls

# show all hidden images
docker images -a
docker image ls -a

# pulling an image without creating a container
docker pull nginx:1.14-alpine

# delete an images
docker rmi <image-reference>

# remove all images
docker rmi $(docker images -a -q)

# run a container containing an image, redis for example
docker run redis

# stop an app, the redis app
docker stop redis

# run app on the background
docker run -d redis

# naming a container
docker run --name <container_name> redis
```



## A Bit Less Basic Commands

```bash
# run a specific version of an image using its tag, here it's redis version 4.0
docker run redis:4.0

# run the lastest version of redis
docker run redis:latest

# in case of an app with inputs, the i means interactive and t for pseudo terminal
docker run -i <image_app>
docker run -it <image_app>

# port mapping, to redirect an internal docker port to another external port (like pipelining)
docker run -p 80:5000 <image_app>

# volume mapping
docker run -v /local/folder:/container/folder

# inspect general information about a container, displayed as a json file
docker inspect <reference_to_container>

# view logs for an app running on background
docker logs <container_reference>
```



## Environment Variables

```bash
# set an env variable for a container
docker run -e APP_COLOR=blue <image_app>
```

The `inspect` command gives all defined environment variables on the container.



## Creating a Docker Image

First you've got to create a file named `Dockerfile`, that will contain the list of commands for Docker to use so it can build your image.

```dockerfile
# base image to use, here it is an ubuntu distribution
FROM Ubuntu

# running shell commands, to update the OS and install python
RUN apt-get update
RUN apt-get install python

# installing flask and 
RUN pip install flask
RUN pip install flask-mysql

# Copying all files (at the same level as Dockerfile) to a folder inside the container
COPY . /opt/source-code

# Default execution when you run this image as a container
ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```

To build your image you have to run this command : `docker build . -t <tag-to-give-to-this-image>`



## CMD vs ENTRYPOINT

Both `CMD` and `ENTRYPOINT` makes containers "executable", which means when you run an app it will execute a certain command by default without any entries form the user. As shown in the [previous section](creating-adocker-image), when running the container it will launch Flask. That's been said, there is a subtle difference between those two : `CMD` can be overridden and `ENTRYPOINT` stays rigid.



## Networking

There is 3 types of networks for Docker containers : 

- **Bridge** - default one - private internal network
- **None** - no network - isolated
- **Host** - linking containers with the host’s network - in this case no need for port mapping

```bash
# to associate the container with a certain network :
docker run ubuntu --network=none
docker run ubuntu --network=host

# create your own internal network 
docker network create --driver bridge --subnet 182.18.0.0/16 custom-isolated-network

docker network create --driver bridge --subnet 182.18.0.1/24 --gateway 182.18.0.1 wp-mysql-network

docker run --network=wp-mysql-network -e DB_Host=mysql-db -e DB_Password=db_pass123 -p 38080:8080 --name webapp --link mysql-db:mysql-db -d kodekloud/simple-webapp-mysql

# for listing all existing networks
docker network ls 

# to inspect information about a certain network
docker network inspect network-name-used
```

Again, the `inspect` Docker command is your friend, it will give all information related to configured networks for a container.

You must know that Docker has a built-in DNS Server, no need to point the IP address, using the container name is sufficient. 

## Docker Storage and file system

```bash
# create a volume
docker volume create data_volume

# running a container linked to this persistent volume, we mount the volume inside the container
docker run -v data_volume:/var/lib/mysql mysql

# if we mount an unexisting volume, docker will create it
docker run -v data_volume2:/var/lib/mysql mysql

# you can also mount any directory on the host to the container
docker run -v /data/mysql:/var/lib/mysql mysql

# a verbose way to do the same thing
docker run \
	--mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```



## Docker Compose

Creating one container can be quite ok and straight forward using command lines. But imagine you want to create a whol system with many containers interacting with each other and having some of them on separate networks, using command lines here and get messy and very tedious. So for this use case, you have to create a YAML file describing the "whole system" that Docker has to create for you!

This YAML file has to be named `docker-compose.yml`, and it goes something like this :

```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

In this (very simple) example, we are instructing Docker to create two containers one called `web` (wich will user a local `Dockerfile` to create an image to be used) and the other one is a redis app based on an existing image `redis:alpine`. In addition to that, we can see a port redirection between the container and the host which will definitely be used to access a web page using this port.

