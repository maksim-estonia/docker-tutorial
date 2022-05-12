# Docker

[Source](https://www.youtube.com/watch?v=YFl2mCHdv24)

`docker-tutorial/docker`

## Building

- build the docker image

```
docker build -t hello-world .
```

- `-t`: tag the image as `hello-world`
- `.`: build using the `Dockerfile` in the current directory

## Running

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

# Docker Compose

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

