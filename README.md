# Docker


## Content

[Installation](#installation)

[Basic commands](#basic-commands)

[Images commands](#images-commands)

[Container comands](#container-comands)

[Networks commands](#networks-commands)

[Manage volumes](#manage-volumes)

[Long commands](#long-commands)

[Docker images](#docker-images)

[Dockerignore](#dockerignore)

[DockerHub](#dockerhub)

[Docker compose](#docker-compose)

[Dockerd settings(daemon)](#dockerd-settingsdaemon)

[Dockerfiles for languages](#dockerfiles-for-languages)

[Python](#python)

[Java](#java)

[PHP](#php)

[Ruby on Rails](#ruby-on-rails)

[Go](#go)

[Logs](#logs)

[Registry](#registry)

[Security](#security)

[Docker alternatives](#docker-alternatives)

[Manage a swarm](#manage-a-swarm)

[Manage stacks](#manage-stacks)


## Installation

### Linux

https://docs.docker.com/engine/install/ubuntu/
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get

In case  Got permission denied from idea terminal

- Example 1: Got permission denied while trying to connect to the Docker daemon socket
sudo chmod 666 /var/run/docker.sock

- Example 2: Server: ERROR: Got permission denied while trying to connect to the Docker daemon socket
sudo newgroup docker
sudo chmod 666 /var/run/docker.sock
sudo usermod -aG docker ${USER}

- Example 3: dial unix /var/run/docker.sock: connect: permission denied

sudo setfacl --modify user:<user name or ID>:rw /var/run/docker.sock

- Example 4: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock:
newgrp docker

[Content](#content)

### Windows

To install without a license you should install release 4.9.0 https://docs.docker.com/desktop/release-notes/ and then register on DockerHub where after confirming @mail you will be able to download the updater.

### Mac

as in windows

[Content](#content)

## Basic commands

`docker --version` show version

`docker-compose --version` show version

[Content](#content)

## Images commands


`docker search <name>` search image in Registry

`docker images`  show images

```
docker search nginx
```

`docker pull <name>` download image from Registry

`docker build <path/to/dir>` build image

`docker image build -t <image name> .` build an image with a tag (note the dot!) current directory

`docker image push <image name>` publish an image to dockerhub

`docker image tag <image id> <tag name>` tag an image - either alias an exisiting image or apply a :tag to one

`docker rmi $(docker images -q)` remove all images

`docker rmi <image name>` remove image


[Content](#content)

## Container comands

`docker run <name>` run container from image

`docker run nginx`

`docker run -d nginx`  run container in detouch mode(background)

`docker run -d  --name my-app nginx`   run container in detouch mode(background) with our name my-app

`docker run -p <public port>:<container port> <image name>`

`docker run -d  -p 8080:80 nginx` run container in detouch mode(background) with publish inner port 80 to out port 8080


`docker run -v ${PWD}:usr/share/nginx/html -p 8080:80 -d nginx` run container in detouch mode(background) with mounting our file


`docker run -it busybox` run in interactive terminal(means you in container)

`docker container exec -it <container id> <command>` run a command in a container

`docker exec -it container_id bash` run in interactive terminal exec - means run process in worked container; bash - name of process

`docker attach container_id` connect to container `container_id`

`ctrl+C ` exit
`ctrl+P -next- ctrl+Q`  exit without stop container
### !!!NOTE!!! 

Each container has own IPAddress you can check it with `inspect` command or when you in (using `docker run -it busybox`) use `hostname -i`


`docker rm <name>` delete container

`docker run -it --rm busybox` run in interactive terminal and delete container after exit

`docker container prune` delete all stoped containers

```docker rm `docker ps -a -q` ```delete all containers 

`docker ps -a -q ` 

`docker rm $(docker pa -aq)`

`docker ps` || `docker container ls` show container list


`docker ps` show work containers

`docker ps -a` show all containers

`docker ps --filter "name=hashicorp-learn"`

`docker logs <name>` show container's logs

`docker logs my-app` 

`docker container logs -f <container id>` Follow the log (STDIN/System.out) of the container

`docker container commit -a "author" <container id> <image name>` Take a snapshot image of a container

`docker start/stop/restart <name>` action with container


`Ctrl+C` exit from container

`docker inspect my-app`  show info about container

`docker inspect my-app | grep IPAddress curl <your_ip_address>` find container's IPAddress and curl it

`docker stop my-app && docker rm my-app && docker rmi nginx`   stop and remove container and image

`docker inspect <container id> --format='{{.State.ExitCode}}'` 

`docker inspect --format='{{.State.Running}}'`

[Content](#content)

## Networks commands

`netstat -nlp`  opened ports from server

`docker-machine ip` Find the IP address of your VirtualMachine, required for Docker Toolbox users only

`docker network ls` list all networks

`docker network create <network name>` create a network using the bridge driver

`docker network ls` - show networks

`docker network inspect bridge -f '{{range .Containers}}{{println .Name}}{{println .IPv4Address}}{{end}}'` - show ip of containers

`docker network create --driver bridge alpine-net` - create our network `alpine-net`

`docker network inspect alpine-net` - inspect network

`docker run -dit --name alpine1 --network alpine-net alpine ash` - run container and connect to our network

`docker network rm alpine-net` - remove network

safe way 

CREATE NEW CUSTOM BRIDGE NETWORK
`docker network create elasticsearch`

CREATE ELASTICSEARCH CONTAINER
```
docker run \
    --network elasticsearch \
    --name elasticsearch \
    -e "discovery.type=single-node" \
    -p 9200:9200 \
    elasticsearch:7.6.2
```

CREATE CURL CONTAINER
```
docker run \
    -it \
    --network elasticsearch \
    --name curl \
    appropriate/curl sh
```

[Content](#content)

## Manage volumes

https://docs.docker.com/storage/volumes/

`docker volume ls` list all volumes 

`docker volume prune` delete all volumes that are not currently mounted to a container

`docker volume inspect <volume name>` inspect a volume (can find out the mount point, the location of the volume on the host system)

`docker volume rm <volume name>` remove a volume

`docker run -v ${PWD}:/usr/share/nginx/html nginx` run nginx with our volume

`run` - creating and running container

`-v ${PWD}:/usr/share/nginx/html` - mount volume with our `index.html`

`${PWD}` - absolute path to current local folder

```
FROM nginx

COPY index.html /usr/share/nginx/html
ADD custom.conf /etc/nginx/conf.d/

EXPOSE 80

ENV FOO bar

CMD ["nginx", "-g", "daemon off;"]

```

`COPY index.html /usr/share/nginx/html` - we add our `index.html`

`ADD custom.conf /etc/nginx/conf.d/` - we add our config `custom.conf`

```yaml
version '3'

services:
  app:
    image: nginx:alpine
    ports:
      - 80:80
    volumes:
      - /var/opt/my_website/dist:/usr/share/nginx/html:ro
```

Syntax: `/host/path:/container/path`

[Content](#content)

#### Differences between `-v` and `--mount` behavior

https://docs.docker.com/storage/volumes/

As opposed to bind mounts, all options for volumes are available for both `--mount` and `-v` flags.

When using volumes with services, only `--mount` is supported.

`docker run -v ${PWD}:usr/share/nginx/html -p 8080:80 -d nginx` run container in detouch mode(background) with mounting our file

```
docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html \
  nginx:latest
```

```
docker run -d \
  --name=nginxtest \
  -v nginx-vol:/usr/share/nginx/html \
  nginx:latest
```

[Content](#content)

#### Use a read-only volume

```
docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html,readonly \
  nginx:latest

```
```
docker run -d \
  --name=nginxtest \
  -v nginx-vol:/usr/share/nginx/html:ro \
  nginx:latest
```

[Content](#content)

#### Windows

`docker run -it -v c:\Data:c:\shareddata microsoft/windowsservercore powershell`

`docker run -it -v hostdata:c:\shareddata microsoft/windowsservercore powershell`  this create volume

`docker volume inspect hostdata` we can find physical location on hdd(Mountpoint)
 
 
`Invoke-Item value_mountpoint` look into directory  powershell

`New-Item -Name Test.txt -ItemType File` create file powershell

`ctrl-p-q`

[Content](#content)


## Long commands


```
docker run \
--name my-app \
-v ${PWD}:usr/share/nginx/html \
 -p 8080:80 \
 -d \
 --rm \
 nginx
```
---




`docker run --name my-database -d -e MYSQL_ROOT_PASSWORD=pass -p 3306:3306 mysql:5.7`  run with setting environment variable `MYSQL_ROOT_PASSWORD=pass`


`docker stop test_long && docker rm $(docker ps -a -q) && docker rmi $(docker images -q)` stop container test_long then remove all containers and images

`docker rmi $(docker images -q)` means remove images  and `$(docker images -q)` this command find id's all images

[Content](#content)

## Docker images

Dockerfile example https://docs.docker.com/engine/reference/builder/#from

```
FROM debian

COPY . /opt/

RUN apt-get update
RUN apt-get install -y nginx

COPY custom.conf /etc/nginx/conf.d/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

`docker build .`  build image (should run from directory where Dockerfile is)

`docker build -t nginx .`  build image with name `nginx` (should run from directory where Dockerfile is)

-----
Our image is like a cake each command is a layer(consume time and memory) --> we should create as less as possible layers

-----

`FROM` - base image

`COPY` - copy files 1st-from 2d-to 

`COPY . .` - from current folder in workdir image

`RUN` - run command for base image

`EXPOSE` - open port

`CMD` - command after building image

`CMD ["python", "main.py"]` run process `python` with argument `main.py`

`ENV` - specify environment variable

`ARG` - arguments wich we can use when build image 

`WORKDIR` - create directory in image

`WORKDIR /app` - create directory `app`

[Content](#content)

### Improving image


```
FROM debian

COPY . /opt/

RUN apt-get update && apt-get install -y \
nginx \
&& rm -rf /var/lib/apt/lists/*

COPY custom.conf /etc/nginx/conf.d/

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Here we optimized:
1. Two commands `RUN apt-get update && apt-get install -y \
nginx \`

2. We can add additional instalation
`RUN apt-get update && apt-get install -y \
nginx \
php \`(php)

3. We add additional command to delete from cache storage (apt-get) 
`&& rm -rf /var/lib/apt/lists/*`

4. If we use alpine we spend less size but we also need to change packet manager. When use iamge alpine or nginx better use one version

5. To build faster images we should have right order. On latest place we should the most changable layers because docker use cache for unchangable layers.

```
FROM alpine:3.11.5

ENV NGINX_VERSION 1.20.2-r0

RUN apk add --no-cache nginx=${NGINX_VERSION} && mkdir -p /run/nginx

EXPOSE 80

COPY custom.conf /etc/nginx/conf.d/

COPY . /opt/

CMD ["nginx", "-g", "daemon off;"]

```

[Content](#content)

## Dockerignore

File to not include in image some files



### `.dockerignore`  example

```
.git
Dockerfile
.pdf
```

[Content](#content)

## DockerHub

`docker hub`

`docker login`			

`docker logout`	

`docker tag imageName username/imageName`	add name

`docker push username/imageName`	publish image

[Content](#content)

## Docker compose

docker-compose.yml
```
version: '3'

services:
  front:
    build: ./front
    restart: always
    ports:
      - '3000:3000'
    volumes: # local volume
      - /app/node_modules # use(save) this folder from docker image
      - ./front:/app    # replace with local files(for development) 
  api:
    build: ./api
    restart: always
    ports:
      - '5555:5000'
    depends_on: # api won't work without mysql
      - mysql
    volumes:
      - /app/node_modules
      - ./api:/app
    environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PORT: '3306'
      MYSQL_PASSWORD: password
      MYSQL_DB: time_db
  mysql:
    image: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: time_db
    volumes: # volume from dockerhost
      - mysql_data:/var/lib/mysql # use volume from dockerhost
  adminer:
    image: adminer
    restart: always
    ports:
      - '8888:8080'

volumes: # create volume on dockerhost
  mysql_data:
```

`docker-compose build` -  build project

`docker-compose up -d` - run project in background

`docker-compose -f docker-compose.public.yml up -d` - run project from file name which  not`docker-compose.yml`  in background

`docker-compose up -d --build` - run project in background and rebild all services

`docker-compose down` - stop project and delete containers

`docker-compose -f docker-compose.public.yml down` - stop project with special name

`docker-compose logs -f service_name` - view service logs

`docker-compose ps` - show container list

`docker-compose exec service_name name_command` - run command

`docker-compose images` - images list

[Content](#content)

### version

!!!!!!!!!!!!! In 3d versio some function like managing volumes, resources(cpu), depends_on condition moved to swarm.

!!!! if you want to use them use 2d version.

[Content](#content)

## Dockerd settings(daemon)

### run keys

`-b, --bridge string`- connect container's network to bridge

`-D, --debug` - turn on debug mode

`-G, --group string` - group for unix socet (by default "docker")

`-H , --host list` - socets for connect

`-l, --log-level string` - log level (by default "info")

`-p, --pidfile string` - path to PID file (by default "var / docker.pid")

`-s, --storage-driver string` - select storage's driver

`-v, --version` - show app's version

https://docs.docker.com/engine/reference/commandline/dockerd/

### environment variables

`DOCKER_DRIVER` -The graph driver to use. 

`DOCKER_NOWARN_KERNEL_VERSION` - Prevent
warnings that your Linux kernel is unsuitable for Docker. 

`DOCKER_RAMDISK` - If set this will disable ‘
pivot_root 

`DOCKER_TMPDIR` - Location for temporary Docker files.

`MOBY_DISABLE_PIGZ` - Do not use
unpigz to decompress layers in parallel when pulling
images, even if it is installed.


### daemon.json location

Linux
`etc/docker/daemon.json`

Windows
`%programdata%\docker\config\daemon.json `,
or can be setting up using Docker Desktop;

MacOS
`etc/docker/daemon.json` hidden in VM,
can be setting up using Docker Desktop.

[Content](#content)

## Dockerfiles for languages

### Python

 gunicorn:

 - gunicorn worker tmp dir /dev/shm move tmp to RAM;
 - gunicorn workers=2 threads=4 worker class=gthread two workers, if thetre is a chance for slow responses;
- gunicorn log file= write to STDOUT/STDERR.

```
FROM python:3.8 AS builder
COPY requirements.txt .

# Installing dependencies to local folder
# (example /root/.local)
RUN pip install --user -r requirements.txt

# second step, without name
FROM python:3.8-slim
WORKDIR /code

# copy dependencies from image,
# which we build on first step
COPY --from=builder /root/.local/bin /root/.local
COPY ./src .

# Defining environment variables
ENV PATH=/root/.local:$PATH
ENV PYTHONUNBUFFERED=1
CMD [ "python", "./server.py" ]
```

```
FROM python:3.8-slim buster

# Defining environment variables for
"activation" virtual environment
ENV VIRTUAL_ENV=/opt/venv
RUN python3 -m venv $VIRTUAL_ENV
ENV PATH="$VIRTUAL_ENV/bin:$PATH"
ENV PYTHONUNBUFFERED=1

# Installing dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Run app, defined variables still work
COPY myapp.py .
CMD ["python", "myapp.py"]
```
[Content](#content)

### Java

Java till 8u121 didn't know about cgroups limitation. Now all OK
Heap limits (Xmx, XX:MaxRAMPercentage… )give to container  25% more


```
# First step
FROM maven:3.6-jdk-8-alpine AS builder
ARG USER_HOME_DIR="/root"
VOLUME "$USER_HOME_DIR/.m2"
ENV MAVEN_CONFIG "$USER_HOME_DIR/.m2"
WORKDIR /app

# Copy dependencies and install them
COPY pom.xml .
RUN mvn -e -B dependency:resolve

# build jar-file
COPY src ./src
RUN mvn -e -B package

#Second step copy app and run it
FROM openjdk:8-jre-alpine
COPY --from=builder /app/target/app.jar /
CMD ["java", "jar", "/app.jar"]
```
[Content](#content)

### PHP

```
FROM php:7.2-fpm
COPY composer.lock composer.json /var/www/
WORKDIR /var/www
RUN apt-get update && apt get install -y \
build-essential \
libpng-dev \
libjpeg62-turbo-dev \
libfreetype6-dev \
locales \
zip \
jpegoptim optipng pngquant gifsicle \
vim \
unzip \
git \
curl
RUN apt-get clean && rm -rf /var/lib/apt/lists/*
RUN docker-php-ext-install pdo_mysql mbstring zip
exif pcntl
```

```
RUN docker-php-ext-configure gd --with-gd --with-
freetype-dir=/usr/include/ --with-jpeg-
dir=/usr/include/ --with-png-dir=/usr/include/
RUN docker-php-ext-install gd
RUN curl -sS https://getcomposer.org/installer |
php -- --install-dir=/usr/local/bin --filename=composer

RUN groupadd -g 1000 www
RUN useradd -u 1000 -ms /bin/bash -g www www
COPY . /var/www
COPY --chown=www:www . /var/www
USER www
EXPOSE 9000
CMD ["php-fpm"]
```

[Content](#content)

### Ruby on Rails

```
FROM ruby:2.3-alpine

RUN set -ex && apk add --update --virtual runtime-deps postgresql-client nodejs libffi-dev readline sqlite && rm -rf /var/cache/apk/*

WORKDIR /tmp
COPY Gemfile Gemfile.lock ./
RUN set -ex && apk add --virtual build-deps build-base openssl-dev postgresql-dev
libc-dev linux-headers libxml2-dev libxslt-dev readline-dev && \
bundle install --clean --no-cache --without development --jobs=2 && \
rm -rf /var/cache/apk/* && \
apk del build-deps
ENV APP_HOME /app
COPY . $APP_HOME
WORKDIR $APP_HOME
ENV RAILS_ENV=production RACK_ENV=production
EXPOSE 3000
CMD ["bundle", "exec", "puma", "-C", "config/puma.rb"]
```

[Content](#content)

### Go

```
# First step
FROM golang:alpine AS builder
# install git, we need to install dependencies
RUN apk update && apk add --no-cache git
ENV USER=appuser
ENV UID=10001

# Create user in the most secured way
RUN adduser \
--disabled-password \
--gecos "" \
--home "/nonexistent" \
--shell "/sbin/nologin" \
--no create-home \
--uid "${UID}" \
"${USER}"WORKDIR $GOPATH/src/mypackage/myapp/

# copy code to image and install dependencies
COPY . .

# If use Go 1.10 and smaller
# RUN go get -d -v

# If use Go 1.11 and higher
RUN go mod download
RUN go mod verify

# Run optimized build
RUN GOOS=linux GOARCH=amd64 go build -ldflags="-w -s" -o /go/bin/hello

# Second step
FROM scratch
# copy all nececary from first step
COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /etc/group /etc/group
COPY --from=builder /go/bin/hello /go/bin/hello

# Simple user
USER appuser:appuser
# Port with number more than 1024
EXPOSE 9292
ENTRYPOINT ["/go/bin/hello"]
```

[Content](#content)

## Logs

`docker logs container_id`  show logs

`docker logs -f container_id`   connect

`docker logs --tail 10 container_id`   10 last records

`docker logs --tail 10 -t container_id`   10 last records + time

`docker logs -ft container_id`   output woth time

```docker run --name logs -it --rm nginx bash```

Additional exporters

https://github.com/draganm/missing container metrics
https://github.com/google/cadvisor

#### Turn on metrics in Docker
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
We need to add link[### daemon.json location]

```json
    "experimental": true,
    "metrics-addr":"0.0.0.0:9100",
```

to daemon.json

Result file:

```json
{
    "experimental": true,
    "metrics-addr":"0.0.0.0:9100",
    "bip": "10.5.0.1/16",
    "default-address-pools":[
      {
        "base":"192.168.0.0/16",
        "size":28
      }
    ],
    "registry-mirrors": ["http://172.20.100.52:5000"]
}
```

See metrics:
```curl localhost:9100/metrics```

#### See resurce consuming by container

```docker stats```

More detailed metrics:

```bash
/sys/fs/cgroup/memory/docker/<longid>/memory.stat
/sys/fs/cgroup/cpu/docker/<longid>/cpuacct.stat
```

We also can use ctop


#### Docker inspect can show the reason why container failed

```docker inspect 5476 |jq '.[0] | {State}'```

[Content](#content)

## Registry

Nexus from Sonatype
Portus from Suse
Quay from RedHat
Artifactory from JFrog
Harbor from VMware

[Content](#content)

## Security

1. Host OS should have good passwords, minimum privileged users, minimum software and SELinux/AppArmor.

2. Update all software

3. Don't use root in container

change root to user

```
FROM ubuntu
RUN mkdir /app
RUN groupadd -r user && useradd -r -s /bin/false -g user
user
WORKDIR /app
COPY . /app
RUN chown -R user:user /app
USER user
CMD node index.js
```

4. Don't use unknown images (look into Dockerfile and build by yourself)

5. Use analysers 
 
  - clair(static opensource)
  - falco(dynamic opensource)
  - snyk 

6. Use linter

- Hadolint

7. Capabilities 

don't run in privileged mode image because we can run sudo command on host

`docker run image --priveleged` = `sudo wget -O - http://сайт/скрипт.sh | bash`

min capabilities:

`docker run -d --cap-drop=all --cap-add=setuid --cap-add=setgid fedora`

8. Min image
Distroless
From: scratch
From: alpine/debian

gcr.io/distroless/static debian10
gcr.io/distroless/base debian10
gcr.io/distroless/java debian10
gcr.io/distroless/cc debian10
gcr.io/distroless/nodejs debian10

https://github.com/GoogleContainerTools/distroless

```
# Start by building the application.
FROM golang:1.13-buster as build

WORKDIR /go/src/app
ADD . /go/src/app
RUN go get -d -v ./...
RUN go build -o /go/bin/app

# Now copy it into our base image.
FROM gcr.io/distroless/base-debian10
COPY --from=build /go/bin/app /
CMD ["/app"]
```

9. Trusted containers

`sudo export DOCKER_CONTENT_TRUST=1`

10. SSHD

!!! Don't use  sshd in container!

Actually when we work in develop we can use different tools and images to debug. But in prod we should use all minimal to prevent vulnerabilities.

[Content](#content)

## Docker alternatives

- FreeBSD Jail

- LXC/LXD

- Podman  https://habr.com/ru/post/659049/

- Crun

- Kata Kontainers

- Firecracker

- RKT

- Docker Enterprise

- Cri-o

[Content](#content)

## Manage a swarm

`docker swarm init (--advertise-addr <ip address>)` Switch the machine into Swarm mode. We didn't cover how to stop swarm mode: `docker swarm leave --force` 

`docker service create <args>`  Start a service in the swarm. The args are largely the same as those you will have used in docker container run.

`docker network create --driver overlay <name>`  Create a network suitable for using in a swarm.

`docker service ls` List all services

`docker node ls` List all nodes in the swarm

`docker service logs -f <service name>` Follow the log for the service. This feature is a new feature in Docker and may not be available on your version (especially if using Linux Repository Packages).

`docker service ps <service name>` List full details of the service - in particular the node on which it is running and any previous failed containers from the service.

`docker swarm join-token <worker|manager>` Get a join token to enable a new node to connect to the swarm, either as a worker or manager.

[Content](#content)

## Manage stacks

`docker stack ls` list all stacks on this swarm.

`docker stack deploy -c <compose file> <stack name>` deploy (or re-deploy) a stack based on a standard compose file.

`docker stack rm <stack name>` delete a stack and its corresponding services/networks/etc. 

[Content](#content)
