# Docker tutorial

## Youtube series

[Source](https://www.youtube.com/watch?v=YFl2mCHdv24)

`docker-tutorial/docker`

### Building

- build the docker image

```
docker build -t hello-world .
```

- `-t`: tag the image as `hello-world`
- `.`: build using the `Dockerfile` in the current directory

### Running

- run the docker container

```
docker run -p 80:80 hello-world
```

- `-p 80:80`: publish a container's port to the host (`hostPort:containerPort`)
- `hello-world`: run the `hello-world` image

Webpage hosted by container can now be accessed by going to `localhost`

```
docker run -p 80:80 -v /home/maksim/docker-tutorial/docker:/var/www/html hello-world
```

- `-v`: Bind mount a volume(`host-src`:`container-dest`, see [doc](https://docs.docker.com/engine/reference/run/#volume-shared-filesystems))

### Docker Compose

[Source](https://www.youtube.com/watch?v=Qw9zlE3t8Ko)

`docker-tutorial/product`

`docker-compose.yml`:

```yml
version: '3'

services:
  product-service:
    build: ./product
    volumes:
      - ./product:/usr/src/app
    ports:
      - 5001:80
  website:
    image: php:apache
    volumes:
      - ./website:/var/www/html
    ports:
      - 5000:80
    depends_on:
      - product-service
```

```
docker-compose up (-d)
```

- `-d`: run in detached mode

to see running docker containers;

```
docker ps
```

to stop running docker containers:

```
docker-compose stop
```

## Docker docs

[Source](https://docs.docker.com/get-started/overview/)

![docker-architecture](docker-docs/images/architecture.png)

The **Docker daemon** (`dockerd`) listens for Docker API requests and manages Docker objects such as images, containers, networks and volumes. A daemon can also communicate with other daemons to manage Docker services. 

The **Docker client** (`docker`) is the primary way that many Docker users interact with Docker. When you use commands such as `docker run`, the client sends these commands to `dockerd`, which carries them out. The `docker` command uses the Docker API. The Docker client can communicate with more than one daemon.

**Docker Desktop** is an easy-to-install application for your Mac or Windows environment that enables you to build and share containerized applications and microservices. Docker Desktop includes the Docker daemon (`dockerd`), the Docker client (`docker`), Docker Compose, Docker Content Trust, Kubernetes, and Credential Helper.

A **Docker Registry** stores Docker Images. Docker Hub is a public registry that anyone can use, and Docker is configured to look for images on Docker Hub by default. You can even run your own private registry.

When you use the `docker pull` or `docker run` commands, the required images are pulled from your configured registry. When you use the `docker push` command, your image is pushed to your configured registry.

### Docker objects

Using Docker, you are creating and using images, containers, networks, volumes, plugins and other objects. 

An **image** is a read-only template with instructions for creating a Docker container. Often, an image is _based on_ on another image, with some additional customization. For example, you may build an image which is based on the ubuntu image, but installs the Apache web server and your application, as well as the configuration details needed to make your application run.

A **container** is a runnable instance of an image. You can create, start, stop, move, or delete a container using the Docker API or CLI. You can connect a container to one or more networks, attach storage to it, or even create a new image based on its current state. 

### Part 1: Getting started

- Build and run an image as a container
- Share Images using Docker Hub
- Deploy Docker applications using multiple containers with a database
- Running applications using Docker Compose

### Part 2: Sample application

1. Create a file named `Dockerfile` 

```dockerfile
# syntax=docker/dockerfile:1
FROM node:12-alpine
RUN apk add --no-cache python2 g++ make
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000
```

```
docker run -d -p 80:80 docker/getting-started
```

2. Build the container image 

```
docker build -t getting-started .
```

- once the alpine image was downloaded, we copied our application and used `yarn` to install our application's dependencies. The `CMD` directive specifies the default command to run when starting a container from this image.
- `-t` flag tags our image. Since we named the image `getting-started`, we can refer to that image when we run a container.
- `.` at the end tells that Docker should look for the `Dockerfile` in the current directory

3. Start an app container

```
docker run -dp 3000:3000 getting-started
```

### Part 3: Update the application

Remove a container using the CLI

1. Get the ID of the container by using the `docker ps` command.

```
docker ps
```

2. Use the `docker stop` command to stop the container.

```
docker stop <the-container-id>
```

3. Once the container has stopped, you can remove it by using the `docker rm` command

```
docker rm <the-container-id>
```

Note: you can stop and remove a container in a single command by adding the force flag: `docker rm -f <the-container-id>`

### Part 4: Share the application

1. Try running the push command you see on Docker Hub.

```
docker push maksimdrachov/getting-started
```

2. Login to Docker Hub

```
docker login -u maksimdrachov
```

3. Use the `docker tag` command to give the `getting-started` image a new name.

```
docker tag getting-started maksimdrachov/getting-started
```

4. Now try to push again

```
docker push maksimdrachov/getting-started
```

### Part 5: Persist the DB

When a container runs, it uses the various layers from an image for its filesystem. Each container also gets its own "scratch space" to create/update/remove files. Any changes won't be seen in another container, even if they are using the same image.

To see this in action we're going to start two containers and create a file in each. What you'll see is that files created in one container aren't available in another. 

1. Start an `ubuntu` container that will create a file named `/data.txt` with a random number between 1 and 10000.

```
docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"
```

- `bash` - start a bash shell
- `-c` - invoking two commands
  - `shuf -i 1-10000` - picks a single random number and writes it to `/data.txt`
  - `tail -f /dev/null` - watching a file to keep the container running

2. `docker exec` into the container

```
docker exec <container-id> cat /data.txt
```

3. Now, let's start another `ubuntu` container (same image) and we'll see we don't have the same file

```
docker run -it ubuntu ls /
```

