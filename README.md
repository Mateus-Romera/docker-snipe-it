# Docker images for Snipe-IT application

This repository contains docker images for the [Snipe-IT application, a free open source IT asset/license management system](https://github.com/snipe/snipe-it).

This work is heavily inspired on docker official images.

## Tags

For now, we only have a single image based on php-fpm official image.

- [`latest`, `5.3.3-fpm-alpine`, `5.3-fpm-alpine`, `5-fpm-alpine`, `fpm-alpine`, `5.3.3-php7.4-fpm-alpine`, `5.3.3-php7.4-fpm-alpine-r0`, `5.3-php7.4-fpm-alpine`, `5-php7.4-fpm-alpine`, `php7.4-fpm-alpine`](https://github.com/Mateus-Romera/docker-snipe-it/blob/main/php7.4/fpm-alpine/Dockerfile)
- [`5.3.2-fpm-alpine`, `5.3-fpm-alpine`, `5-fpm-alpine`, `fpm-alpine`, `5.3.2-php7.4-fpm-alpine`, `5.3.2-php7.4-fpm-alpine-r0`, `5.3-php7.4-fpm-alpine`, `5-php7.4-fpm-alpine`, `php7.4-fpm-alpine`](https://github.com/Mateus-Romera/docker-snipe-it/blob/main/php7.4/fpm-alpine/Dockerfile)
- [`5.3.1-fpm-alpine`, `5.3-fpm-alpine`, `5-fpm-alpine`, `fpm-alpine`, `5.3.1-php7.4-fpm-alpine`, `5.3.1-php7.4-fpm-alpine-r0`, `5.3-php7.4-fpm-alpine`, `5-php7.4-fpm-alpine`, `php7.4-fpm-alpine`](https://github.com/Mateus-Romera/docker-snipe-it/blob/main/php7.4/fpm-alpine/Dockerfile)
- [`5.3.0-fpm-alpine`, `5.3-fpm-alpine`, `5-fpm-alpine`, `fpm-alpine`, `5.3.0-php7.4-fpm-alpine`, `5.3-php7.4-fpm-alpine`, `5-php7.4-fpm-alpine`, `php7.4-fpm-alpine`](https://github.com/Mateus-Romera/docker-snipe-it/blob/main/php7.4/fpm-alpine/Dockerfile)

## Usage

```
$ docker pull pirasbro/snipe-it:latest
```

## Docker Secrets

As an alternative to passing sensitive information via environment variables, `_FILE` may be appended to the previously listed environment variables, causing the initialization script to load the values for those variables from files present in the container. In particular, this can be used to load passwords from Docker secrets stored in `/run/secrets/<secret_name>` files. For example:

```console
$ docker run --name snipe-it -e APP_KEY_FILE=/run/secrets/app-key ... -d pirasbro/snipe-it:tag
```

Currently, this is supported for `APP_KEY`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`, `REDIS_HOST`, `REDIS_PASSWORD`, `REDIS_PORT`, `MAIL_HOST`, `MAIL_PORT`, `MAIL_USERNAME` and `MAIL_PASSWORD`.

## Example

You should get it working easily on a simple environment using `docker-compose` with the files below:

```
.../snipe-it
.../snipe-it/docker-compose.yaml
.../snipe-it/no-ssl.conf.template
```

Example `docker-compose.yaml` for `snipe-it`:

```yaml
volumes:
  snipeit_data:
  web:
  database:

services:
  snipeit:
    image: docker.io/pirasbro/snipe-it:latest
    environment:
      # SnipeIT settings
      APP_URL: localhost:8080
      APP_KEY: base64:CdNywz90QeGfAvL4nw7sqHwr9nI8v4z+yYcWpEgH+4g=
      # Database settings
      DB_HOST: mariadb
      DB_PORT: 3306
      DB_DATABASE: snipeit
      DB_USERNAME: dev
      DB_PASSWORD: dev
      # Session settings
      SECURE_COOKIES: "false"
    volumes:
      - web:/var/www/html
      - snipeit_data:/var/lib/snipeit
    depends_on:
      - mariadb
    restart: on-failure

  nginx:
    image: docker.io/nginx:alpine
    environment:
      NGINX_HOST: localhost
      NGINX_PORT: 8080
      NGINX_UPSTREAM: snipeit:9000
    ports:
      - "8080:8080"
    volumes:
      - web:/var/www/html
      # Nginx default no-ssl config template
      - "./no-ssl.conf.template:/etc/nginx/templates/default.conf.template:ro"
    depends_on:
      - snipeit

  mariadb:
    image: docker.io/mariadb:latest
    user: mysql:mysql
    environment:
      MYSQL_ROOT_PASSWORD: dev
      MYSQL_DATABASE: snipeit
      MYSQL_USER: dev
      MYSQL_PASSWORD: dev
    volumes:
      - database:/var/lib/mysql
```

`no-ssl.conf.template`:

```conf
# ----------------------------------------------------------------------
# | Config file for non-secure $NGINX_HOST host                        |
# ----------------------------------------------------------------------
#
# This file is a template for a non-secure Nginx server.
# This Nginx server listens for the $NGINX_HOST host and handles requests.
# Replace $NGINX_HOST in docker-compose.yml with your hostname before enabling.

# Upstream blocks to abstract backend connection(s):
# Direct connection with sock
upstream default {
    server ${NGINX_UPSTREAM};
}

server {
    # The port the server will listen to
    # listen [::]:${NGINX_PORT};
    listen ${NGINX_PORT};

    # The host name to respond to
    server_name ${NGINX_HOST};

    # Path for static files
    root /var/www/html/public;

    # This should be in your http block and if it is, it's not needed here
    index index.html index.php;

    # Application settings
    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location / {
        # This is cool because no php is touched for static content.
        # include the "?$args" part so non-default permalinks doesn't break when using query string
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        #NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini
        include fastcgi_params;
        fastcgi_intercept_errors on;
        fastcgi_pass default;
        #The following parameter can be also included in fastcgi_params file
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        
        # try_files $uri =404;
        # fastcgi_index index.php;
        # fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
        expires max;
        log_not_found off;
    }
}
```

Run `docker-compose up`, wait for it to initialize completely, and visit `http://localhost:8080`.

## TODO

- get new versions from snipe-it releases automatically;
- create example using docker secrets;
- create docker-swarm example;
- create kubernetes example;
- create CI using github actions for:
    - build new images everyday;
    - use trivy to scan images for CVE;
- create Helm package;
- add images for:
    - fpm-buster;
    - buster (apache);