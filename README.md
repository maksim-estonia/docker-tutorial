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

#### Container volumes

Volumes provide the ability to connect specific filesystem paths of the container back to the host machine. If a directory in the container is mounted, changes in that directory are also seen on the host machine. If we mount that same directory across container restarts, we'd see the same files.

There are two main types of volumes. We will eventually use both, but we will start with **named** volumes.

1. Create a volume by using the `docker volume create` command

```
docker volume create todo-db
```

2. Stop and remove the todo app container once again (`docker rm -f <id>`)

3. Start the todo app container, but add the `-v` flag to specify a volume mount. We will use the named volume and mount it to `/etc/todos`, which will capture all files created at the path.

```
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```

4. Once the container starts up, open the app and add a few items to your todo list.

5. Stop and remove the container for the todo app. (`docker ps` then `docker rm -f <id>`)

6. Start a new container using the same command from above

7. Open the app. You should see your items still in the list

If you want to know where docker stores your data:

```
docker volume inspect todo-db
```

### Part 6: Use bind mounts

With **bind mounts**, we control the exact mountpoint on the host. We can use this to persist data, but it's often used to provide additional data into containers. When working on an application, we can use a bind mount to mount our source code into the container to let it see code changes, respond, and let us see the changes right away.

To run our container to support a development workflow, we will do the following:

- Mount our source code in the container
- Install all dependencies, including the "dev" dependencies
- Start nodemon to watch for filesystem changes

1. Make sure you don't have any previous getting-started containers running

2. Run the following command from the app directory.

```
docker run -dp 3000:3000 \
     -w /app -v "$(pwd):/app" \
     node:12-alpine \
     sh -c "yarn install && yarn run dev"
```

  - `-dp 3000:3000` - detached mode, port mapping
  - `-w /app` - sets the "working directory" or the current directory that the command will run from
  - `-v "(pwd):/app"` - bind mount the current directory from the host in the container into the `/app` directory.
  - `node:12-alpine` - the image to use. Note that this is the base image for our app form the Dockerfile
  - `sh -c "yarn install && yarn run dev"` - the command. We're starting a shell using `sh` (alpine doesnt have `bash`) and running `yarn install` to install all dependencies and then running `yarn run dev`. If we look in the `package.json`, we'll see that the `dev` script is starting `nodemon`. 

3. You can watch the logs using `docker logs -f <container-id>`. 

4. No, let's make a change to the app. In the `src/static/js/app.js` file, let's change the "Add Item" button to simply say "Add"

5. Refresh the page and you should see the change reflected.

6. Make other changes, when you're done, stop the container and build your new image using:

```
docker build -t getting-started .  
```

### Part 7: Multi-container apps

In general, each container should do one thing and do it well.

Remember that container, by default, run in isolation and don't know anything about other processes or containers on the same machine. So, how do we allow one container to talk to another? The answer is networking.

If two containers are on the same network, they can talk to each other. If they aren't, they can't.

There are two ways to put a container on a network: 1) assign it at start or 2) connect an existing container. For now, we will create the network first and attach the MySQL container at startup.

1. Create the network

```
docker network create todo-app
```

2. Start a MySQL container and attach it to the network. 

```
docker run -d \
     --network todo-app --network-alias mysql \
     -v todo-mysql-data:/var/lib/mysql \
     -e MYSQL_ROOT_PASSWORD=secret \
     -e MYSQL_DATABASE=todos \
     mysql:5.7
```

3. To confirm we have the database up and running, connect to the database and verify it connects.

```
docker exec it <mysql-container-id> mysql -u root -p
```

If we run another container on the same network, how do we find the container (remember each container has its own IP address)?

We're going to make use of the `nicolaka/netshoot` container, which ships with a _lot_ of tools that are useful for troubleshooting or debugging networking issues.

1. Start a new container using the netshoot image. Make sure to connect it to the same network:

```
docker run -it --network todo-app nicolaka/netshoot
```

2. Inside the container, we're going to use the `dig` command, which is a useful DNS tool. We're going to look up the IP address for the hostname `mysql`.

```
dig mysql
```

The todo app supports the setting of a few environment variables to specify MySQL connection settings. They are:

- `MYSQL_HOST` - the hostname for the running MySQL server
- `MYSQL_USER` - the username to use for the connection
- `MYSQL_PASSWORD` - the password to use for the connection
- `MYSQL_DB` - the database to use once connected

### Part 8: Use Docker Compose

1. At the root of the app project, create a file named `docker-compose.yml`

2. In the compose file, we'll start off by defining the schema version.

```
version: "3.7"
```

3. Next, we'll define the list of services (or containers) we want to run as part of our application.

```
version: "3.7"

services:
```

#### Define the app service

This was the command we were using to define our app container:

```
docker run -dp 3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:12-alpine \
  sh -c "yarn install && yarn run dev"
```

1. First, we'll define the service entry and the image for the container. We can pick any name for the service. The name will automatically become a network alias, which will be useful when defining our MySQL service.

```
version: "3.7"

services:
  app:
    image: node:12-alpine
```

2. Typically, you will see the `command` close to the `image` definition. 

```
 version: "3.7"

 services:
   app:
     image: node:12-alpine
     command: sh -c "yarn install && yarn run dev"
```

3. Let's migrate the `-p 3000:3000`

```
 version: "3.7"

 services:
   app:
     image: node:12-alpine
     command: sh -c "yarn install && yarn run dev"
     ports:
      - 3000:3000
```

4. Next, we'll migrate both the working directory (`-w /app`) and the volume mapping (`-v "$(pwd):/app"`) by using the `working_dir` and `volumes` definitions. 

```
 version: "3.7"

 services:
   app:
     image: node:12-alpine
     command: sh -c "yarn install && yarn run dev"
     ports:
      - 3000:3000
     working_dir: /app
     volumes:
      - ./:/app
```

5. Finally, we need to migrate the environment variable definitions using the `environment` key.

```
 version: "3.7"

 services:
   app:
     image: node:12-alpine
     command: sh -c "yarn install && yarn run dev"
     ports:
      - 3000:3000
     working_dir: /app
     volumes:
      - ./:/app
     environment:
      MYSQL_HOST: mysql
      MYSQL_USER: root
      MYSQL_PASSWORD: secret
      MYSQL_DB: todos
```

#### Define the MySQL service

The command that we used:

```
docker run -d \
  --network todo-app --network-alias mysql \
  -v todo-mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=todos \
  mysql:5.7
```

1. First define the new service and name it `mysql`

```
 version: "3.7"

 services:
   app:
     # The app service definition
   mysql:
     image: mysql:5.7
```

2. Next, we'll define the volume mapping. When we ran the container with `docker run`, the named volume was created automatically. However, that doesn't happen when running with Compose. We need to define the volume in the top-level `volumes:` section and then specify the mountpoint in the service config. By simply providing only the volume name, the default options are used. 

```
 version: "3.7"

 services:
   app:
     # The app service definition
   mysql:
     image: mysql:5.7
     volumes:
       - todo-mysql-data:/var/lib/mysql

 volumes:
   todo-mysql-data:
```

3. Finally, we only need to specify the environment variables.

```
 version: "3.7"

 services:
   app:
     # The app service definition
   mysql:
     image: mysql:5.7
     volumes:
       - todo-mysql-data:/var/lib/mysql
     environment:
       MYSQL_ROOT_PASSWORD: secret
       MYSQL_DATABASE: todos

 volumes:
   todo-mysql-data:
```

