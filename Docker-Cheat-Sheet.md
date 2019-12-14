# Docker Cheat-Sheet

This cheat-sheet is "highly" using references from the official Docker documentation and the very great introductory [video](https://www.youtube.com/watch?v=zJ6WbK9zFpI) made by [Mumshad Mannambeth](https://github.com/mmumshad).

### This cheat-sheet contains :

- [Installing Docker on Ubuntu 18.04](#installing-docker-on-ubuntu-18-4)

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

## General Commands

```bash
# run a specific version of an image using its tag, here it's redis version 4.0
docker run redis:4.0

# run the lastest version of redis
docker run redis:latest

# in case of app with input, the i means interactive and t for pseudo terminal
docker run -i <image_app>
docker run -it <image_app>

# port mapping, direct an internal port to docker to another external port (pipeline)
docker run -p 80:5000 <image_app>

# volume mapping
docker run -v /local/folder:/container/folder

# inspect containers retur a json
docker inspect <reference_to_container>

# view logs for app running on the background
docker logs <container_reference>
```



## Env Variables

```bash
# set an env variable for a container
docker run -e APP_COLOR=blue <web_app>

# getting env variables is contained on the inspect command under "Env" key
```



## Docker images

Docker file

```
FROM Ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```



`docker build Dockerfile -t <tag-to-give-to-this-image>`



## CMD vs Entrypoint

 Entrypoint configure a container to run as an executable. Using entypoint we give it just the parameters without specifying the CMD in the shell. (need to be clearer)

## Networking

There is 3 types of network for docker containers : 

- Bridge (i think between containers) - default one - private internal network
- None (no network) - isolated
- Host (linking containers with the host’s network) - in this case no need for port mapping

```bash
# To associate the container with a certain network :
docker run ubuntu --network=noneDocker run ubuntu --network=host

# Create your own internal network 
docker network create --driver bridge --subnet 182.18.0.0/16 custom-isolated-network

docker network create --driver bridge --subnet 182.18.0.1/24 --gateway 182.18.0.1 wp-mysql-network

docker run --network=wp-mysql-network -e DB_Host=mysql-db -e DB_Password=db_pass123 -p 38080:8080 --name webapp --link mysql-db:mysql-db -d kodekloud/simple-webapp-mysql

# for listing all commands
docker network ls 

# There is a section in the docker inspect command to check the network configurations for a container

# Docker has a built-in dns server, no need to point the ip address, using the container name is sufficient.
# To inspect information about a certain entwork docker network inspect bridge
```



## Docker Storage and file system

```bash
#create a volume
docker volume create data_volume

# running a container linked to this persistent volume, we mount the volume inside the container
docker run -v data_volume:/var/lib/mysql mysql
#if we mount an unexisting volume, docker will create it
docker run -v data_volume2:/var/lib/mysql mysql

#you can also mount any directory on the host to the container
docker run -v /data/mysql:/var/lib/mysql mysql

# a verbose way to do the same thing
docker run \
	--mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```





Compose
Configuration in yml files