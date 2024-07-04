# Dolibarr docker images

Provides a web and database server images for dolibarr. The dolibarr code itself is not included, you are supposed to adapt the docker-compose.yml file to fit your own paths and ports.

To build the images, run

```sh
docker build -t pythoff/dolibarr-db db/
```
and 

```sh
docker build -t pythoff/dolibarr-web web/
```
Adapt the "volumes" section of the .yml file and you are ready to go.

## The docker-compose.yml file

This YAML configuration file is for Docker Compose, a tool for defining and running multi-container Docker applications. The file specifies a setup for Dolibarr, which is an ERP (Enterprise Resource Planning) and CRM (Customer Relationship Management) system. This configuration consists of two services: `dolibarr-db` and `dolibarr-web`.

Here's a detailed breakdown of the file:

### Version
```yaml
version: '2'
```
Specifies the version of the Docker Compose file format. In this case, it is version 2.

### Services
Defines the services that make up the application.

#### dolibarr-db
```yaml
  dolibarr-db:
    image: pythoff/dolibarr-db:latest
    container_name: dolibarr-db
    hostname: dolibarr-db
    environment:
      - COMPOSE_HTTP_TIMEOUT=120
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=dolibarr
    ports:
      - "13306:3306"
    external_links:
      - dolibarr-web
    volumes:
      - ./var/lib/mysql:/var/lib/mysql
```
- **image**: Specifies the Docker image to use for the service, in this case, `pythoff/dolibarr-db:latest`.
- **container_name**: Names the container `dolibarr-db`.
- **hostname**: Sets the hostname inside the container to `dolibarr-db`.
- **environment**: Sets environment variables inside the container:
  - `COMPOSE_HTTP_TIMEOUT=120`: Increases the HTTP timeout to 120 seconds.
  - `MYSQL_ROOT_PASSWORD=root`: Sets the MySQL root password to `root`.
  - `MYSQL_DATABASE=dolibarr`: Creates a database named `dolibarr`.
- **ports**: Maps port `3306` inside the container to `13306` on the host machine.
- **external_links**: Creates a link to the `dolibarr-web` service.
- **volumes**: Mounts the host directory `./var/lib/mysql` to `/var/lib/mysql` inside the container for persistent storage of MySQL data.

#### dolibarr-web
```yaml
  dolibarr-web:
    image: pythoff/dolibarr-web:latest
    container_name: dolibarr-web
    hostname: dolibarr-web
    depends_on:
      - dolibarr-db
    external_links:
      - dolibarr-db
    volumes:
      # mount code
      - ./htdocs:/var/www/html/
      # apache config
      - ./etc/develop/apache2/000-default.conf:/etc/apache2/sites-enabled/000-default.conf
      # php config
      - ./etc/develop/php/custom.ini:/usr/local/etc/php/conf.d/99-custom.ini
      - ./var/log:/var/www/html/log
    ports:
      - "2222:22"
      - "88:80"
    environment:
      - SSH_ROOT_PASSWORD=root
      - APPLICATION_ENV=${APPLICATION_ENV}
      - XDEBUG_REMOTE_HOST=${XDEBUG_REMOTE_HOST}
      - PHP_IDE_CONFIG=${PHP_IDE_CONFIG}
      - XDEBUG_CONFIG=${XDEBUG_CONFIG}
      - VIRTUAL_HOST=dolibarr.local
```
- **image**: Specifies the Docker image to use for the service, `pythoff/dolibarr-web:latest`.
- **container_name**: Names the container `dolibarr-web`.
- **hostname**: Sets the hostname inside the container to `dolibarr-web`.
- **depends_on**: Specifies that this service depends on `dolibarr-db`, so it should start only after `dolibarr-db` is up and running.
- **external_links**: Creates a link to the `dolibarr-db` service.
- **volumes**: Mounts several host directories to the container:
  - `./htdocs` to `/var/www/html/` for the web application code.
  - `./etc/develop/apache2/000-default.conf` to `/etc/apache2/sites-enabled/000-default.conf` for Apache configuration.
  - `./etc/develop/php/custom.ini` to `/usr/local/etc/php/conf.d/99-custom.ini` for PHP configuration.
  - `./var/log` to `/var/www/html/log` for logging.
- **ports**: Maps ports:
  - `22` inside the container to `2222` on the host for SSH.
  - `80` inside the container to `88` on the host for HTTP.
- **environment**: Sets environment variables inside the container:
  - `SSH_ROOT_PASSWORD=root`: Sets the root password for SSH.
  - `APPLICATION_ENV=${APPLICATION_ENV}`: Uses the `APPLICATION_ENV` variable from the host environment.
  - `XDEBUG_REMOTE_HOST=${XDEBUG_REMOTE_HOST}`: Uses the `XDEBUG_REMOTE_HOST` variable from the host environment.
  - `PHP_IDE_CONFIG=${PHP_IDE_CONFIG}`: Uses the `PHP_IDE_CONFIG` variable from the host environment.
  - `XDEBUG_CONFIG=${XDEBUG_CONFIG}`: Uses the `XDEBUG_CONFIG` variable from the host environment.
  - `VIRTUAL_HOST=dolibarr.local`: Sets the virtual host to `dolibarr.local`.

### Networks
```yaml
networks:
  default:
      name: dolibarr
      external: true
```
- **networks**: Specifies the networks the services will use.
  - **default**: Uses the `dolibarr` network and indicates it is an external network, meaning it is created outside of this Docker Compose file.

### Summary
This Docker Compose file defines a multi-container application with a MySQL database (`dolibarr-db`) and a web server (`dolibarr-web`). It sets up environment variables, port mappings, volume mounts, and network configurations necessary for running Dolibarr.
