Docker web component

`Dockerfile.web`
```
FROM php:8.0-apache
RUN docker-php-ext-install pdo_mysql
```

Docker DB component

`Dockerfile.db`
```
FROM mariadb:10.7
ADD ./db/db_setup.sql /docker-entrypoint-initdb.d/init.sql
```

Spin up the db container

```bash
docker container run -d --name db -e MYSQL_ROOT_PASSWORD=12345 img-db
```

#### Linking

Spin up the web container and link it to the db container

```bash
docker container run -d --name c-web -p 8080:80 -v $(pwd)/web:/var/www/html --link c-db:db img-web
```

The link is expressed in the web container

`c-web /etc/hosts`
```
172.17.0.2      db a212c975ce38 c-db
```

#### Isolated Network

Create an isolated network

```bash
docker network create --driver bridge app-network
```

Spin the containers and link them to the newly created network

```bash
dpcler container run -d --net app-network --name ...
```

There is no record in the `/etc/hosts` file but the containers are linked

#### Docker Compose

https://docs.docker.com/compose/install/ 

Web Dockerfile

`Dockerfile`
```
FROM php:8.0-apache
COPY ./web /var/www/html
```

`docker-compose.yml`
```
services:
  web:
    build: .
    ports:
        - 8080:80
```

Test the file

```bash
docker compose config
```

Build the image with compose

```bash
docker compose build
```

```bash
docker compose {up|down}
```

To start the service in detached mode

```bash
docker compose up -d [--build]
```

`--build`: To rebuild images if the docker-compose file is updated

```bash
docker compose logs
```

Example docker-compose file of web and database services linked together

`docker-compose.yaml`
```
services:
  web:
    build:
        context: .
        dockerfile: Dockerfile.web
    ports:
        - 8080:80
    volumes:
        - "/home/vagrant/bgapp/web:/var/www/html:ro"
    networks:
        - app-network
    depends_on:
        - db

  db:
    build:
        context: .
        dockerfile: Dockerfile.db
    networks:
        - app-network
    environment:
        MYSQL_ROOT_PASSWORD: "12345"
networks:
    app-network:
```

Values can be stored separately in an environment file. Do not upload to GIT!. Add to `.gitignore` file.

`.env`
```
PROJECT_ROOT=/home/vagrant/bgapp/web
DB_ROOT_PASSWORD=12345
```

`docker-compose.yaml`
```
volumes:
        - "${PROJECT_ROOT}:/var/www/html:ro"
...
environment:
    MYSQL_ROOT_PASSWORD: "${DB_ROOT_PASSWORD}"
```


