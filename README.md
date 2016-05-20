# What is WordPress?

> WordPress is one of the most versatile open source content management systems on the market. WordPress is built for high performance and is scalable to many servers, has easy integration via REST, JSON, SOAP and other formats, and features a whopping 15,000 plugins to extend and customize the application for just about any type of website.

https://www.wordpress.org/

# Prerequisites

To run this application you need Docker Engine 1.10.0. Docker Compose is recomended with a version 1.6.0 or later.

# How to use this image

## Run WordPress with a Database Container

Running WordPress with a database server is the recommended way. You can either use docker-compose or run the containers manually.

### Run the application using Docker Compose

This is the recommended way to run WordPress. You can use the following docker compose template:

```
version: '2'

services:
  mariadb:
    image: bitnami/mariadb:latest
    volumes_from:
      - mariadb_data

  mariadb_data:
    image: bitnami/mariadb:latest
    entrypoint: 'true'

  application:
    build: bitnami/wordpress:latest
    ports:
      - 80:80
      - 443:443
    volumes_from:
      - application_data
    depends_on:
      - mariadb

  application_data:
    image: bitnami/wordpress:latest
    volumes:
      - /bitnami/wordpress
      - /bitnami/apache
    entrypoint: 'true'
```

### Run the application manually

If you want to run the application manually instead of using docker-compose, these are the basic steps you need to run:

1. Create a new network for the application and the database:

  ```
  $ docker network create wordpress_network
  ```

2. Start a MariaDB database in the network generated:

  ```
  $ docker run -d --name mariadb --net=wordpress_network bitnami/mariadb
  ```

  *Note:* You need to give the container a name in order to WordPress to resolve the host

3. Run the WordPress container:

  ```
  $ docker run -d -p 80:80 --name wordpress --net=wordpress_network bitnami/wordpress
  ```

Then you can access your application at http://your-ip/

## Persisting your application

If you remove every container all your data will be lost, and the next time you run the image the application will be reinitialized. To avoid this loss of data, you should mount a volume that will persist even after the container is removed. If you are using docker-compose your data will be persistent as long as you don't remove `mariadb_data` and `application_data` containers. Those are data volume containers (See https://docs.docker.com/engine/userguide/containers/dockervolumes/ for more information). If you have run the containers manually or you want to mount the folders with persistent data in your host follow the next steps:

> **Note!** If you have already started using your application, follow the steps on [backing](#backing-up-your-application) up to pull the data from your running container down to your host.

### Mount persistent folders in the host using docker-compose

This requires a sightly modification from the template previously shown:
```
version: '2'

services:
  mariadb:
    image: bitnami/mariadb:latest
    volumes_from:
      - mariadb_data

  mariadb_data:
    image: bitnami/mariadb:latest
    entrypoint: 'true'
    volumes:
      - /your/local/path/bitnami/mariadb/data:/bitnami/mariadb/data
      - /your/local/path/bitnami/mariadb/conf:/bitnami/mariadb/conf

  application:
    image: bitnami/wordpress:latest
    ports:
      - 80:80
    volumes_from:
      - application_data

  application_data:
    image: bitnami/wordpress:latest
    volumes:
      - /bitnami/wordpress
      - /bitnami/apache
    entrypoint: 'true'
    mounts:
      - /your/local/path/bitnami/wordpress:/bitnami/wordpress
      - /your/local/path/bitnami/apache:/bitnami/apache
```

### Mount persistent folders manually

In this case you need to specify the directories to mount on the run command. The process is the same than the one previously shown:

1. If you haven't done this before, create a new network for the application and the database:

  ```
  $ docker network create wordpress_network
  ```

2. Start a MariaDB database in the previous network:

  ```
  $ docker run -d --name mariadb -v /your/local/path/bitnami/mariadb/data:/bitnami/mariadb/data -v /your/local/path/bitnami/mariadb/conf:/bitnami/mariadb/conf --network=wordpress_network bitnami/mariadb
  ```

  *Note:* You need to give the container a name in order to WordPress to resolve the host

3. Run the WordPress container:

  ```
  $ docker run -d -p 80:80 --name wordpress -v /your/local/path/bitnami/wordpress:/bitnami/wordpress --network=wordpress_network bitnami/wordpress
  ```

# Upgrade this application

Bitnami provides up-to-date versions of MariaDB and WordPress, including security patches, soon after they are made upstream. We recommend that you follow these steps to upgrade your container. We will cover here the upgrade of the WordPress container. For the MariaDB upgrade see https://github.com/bitnami/bitnami-docker-mariadb/blob/master/README.md#upgrade-this-image

1. Get the updated images:

```
$ docker pull bitnami/wordpress:latest
```

2. Stop your container

 * For docker-compose: `$ docker-compose stop wordpress`
 * For manual execution: `$ docker stop wordpress`

3. (For non-compose execution only) Create a [backup](#backing-up-your-application) if you have not mounted the wordpress folder in the host.

4. Remove the currently running container

 * For docker-compose: `$ docker-compose rm -v wordpress`
 * For manual execution: `$ docker rm -v wordpress`

5. Run the new image

 * For docker-compose: `$ docker-compose start wordpress`
 * For manual execution ([mount](#mount-persistent-folders-manually) the directories if needed): `docker run --name wordpress bitnami/wordpress:latest`

# Configuration
## Environment variables
 When you start the wordpress image, you can adjust the configuration of the instance by passing one or more environment variables either on the docker-compose file or on the docker run command line. If you want to add a new environment variable:

 * For docker-compose add the variable name and value under the application section:
```
application:
  image: bitnami/wordpress:latest
  ports:
    - 80:80
  environment:
    - WORDPRESS_PASSWORD=my_password
  volumes_from:
    - application_data
```

 * For manual execution add a `-e` option with each variable and value:

```
 $ docker run -d -e WORDPRESS_PASSWORD=my_password -p 80:80 --name wordpress -v /your/local/path/bitnami/wordpress:/bitnami/wordpress --network=wordpress_network bitnami/wordpress
```

Available variables:

 - `WORDPRESS_USERNAME`: WordPress application username. Default: **user**
 - `WORDPRESS_PASSWORD`: WordPress application password. Default: **bitnami**
 - `WORDPRESS_EMAIL`: WordPress application email. Default: **user@example.com**
 - `MARIADB_USER`: Root user for the MariaDB database. Default: **root**
 - `MARIADB_PASSWORD`: Root password for the MariaDB.
 - `MARIADB_HOST`: Hostname for MariaDB server. Default: **mariadb**
 - `MARIADB_PORT`: Port used by MariaDB server. Default: **3306**

# Backing up your application

To backup your application data follow these steps:

1. Stop the running container:

* For docker-compose: `$ docker-compose stop wordpress`
* For manual execution: `$ docker stop wordpress`

2. Copy the WordPress data folder in the host:

```
$ docker cp /your/local/path/bitnami:/bitnami/wordpress
```

# Restoring a backup

To restore your application using backed up data simply mount the folder with WordPress data in the container. See [persisting your application](#persisting-your-application) section for more info.

# Contributing

We'd love for you to contribute to this container. You can request new features by creating an
[issue](https://github.com/bitnami/wordpress/issues), or submit a
[pull request](https://github.com/bitnami/wordpress/pulls) with your contribution.

# Issues

If you encountered a problem running this container, you can file an
[issue](https://github.com/bitnami/wordpress/issues). For us to provide better support,
be sure to include the following information in your issue:

- Host OS and version
- Docker version (`docker version`)
- Output of `docker info`
- Version of this container (`echo $BITNAMI_APP_VERSION` inside the container)
- The command you used to run the container, and any relevant output you saw (masking any sensitive
information)

# License

Copyright 2015 Bitnami

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
