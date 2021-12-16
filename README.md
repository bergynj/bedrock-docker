
# Base Docker Compose and Bedrock (WordPress)

Use WordPress locally with Docker using [Docker compose](https://docs.docker.com/compose/)

## Contents

+ PHP 7.4
+ Custom domain for example `100conversations.local`
+ Custom nginx config in `./docker/services/nginx/conf`
+ Custom PHP `php.ini` config in `./docker/services/php/conf`
+ [Bedrock](https://roots.io/bedrock/) - modern development tools, easier configuration, and an improved secured folder structure for WordPress
+ [WP-CLI](https://wp-cli.org/) - WP-CLI is the command-line interface for WordPress.

## Instructions

+ [Docker](https://www.docker.com/get-started)
+ [mkcert](https://github.com/FiloSottile/mkcert) for creating the SSL cert.

Install mkcert:

```
brew install mkcert
brew install nss # if you use Firefox
```

## Installation steps

Both step 1. and 2. below are required:

#### 1. For Docker and the CLI script (Required step)

Copy `./docker/.env.example` in the project root to `./docker/.env` and edit your preferences.

Example:

```dotenv
APP_NAME=100conv
DOMAIN=100conversations.local
DB_HOST=mysql
DB_NAME=100conv
DB_ROOT_PASSWORD=[choose-db_root-pwd]
DB_USR=phm
DB_USR_PASSWORD=[choose-db_usr-pwd]
DB_TABLE_PREFIX=wp_
```

#### 2. Configuring hostfile and local SSL cert

Make sure your `/etc/hosts` file has a record for used domains.
you should add this below:

```
127.0.0.1   100conversations.local
```

Option 1). Use HTTPS with a custom domain

1b. Create a SSL cert:

```shell
cd docker/scripts/
./create-cert.sh
```
* This will generate SSL under `./docker/certs` folder.

This script will create a locally-trusted development certificates. It requires no configuration.

> mkcert needs to be installed like described in Requirements. Read more for [Windows](https://github.com/FiloSottile/mkcert#windows) and [Linux](https://github.com/FiloSottile/mkcert#linux)

Option 2). Run nginx without SSL

Inside `./docker/services/nginx/conf` there is a simple nginx configuration without SSL (simple-nossl.conf.conf).

If you wish to use it - the simplest way is to copy/rename that file to be the default configuration.

eg.
```shell
cd docker/services/nginx/conf/
cp simple-nossl.conf.conf default.conf.conf
```

### Setting up directories

```console
    $ mkdir -p docker/data/db
    $ chmod -R go+rX docker/data
    $ chmod -R go+rX bedrock
    $ chmod -R g+w bedrock/web/wp/wp-content/
    $ chmod -R g+w bedrock/web/app/plugins/
    $ sudo chown -R ${USER}:www-data bedrock
```

### Configuring environment for docker-compose

Docker-compose uses `.env` automatically. For a development environment,
`docker-compose.yml` contains usable defaults for the same variables.

### Shell configuration

This project is using a makefile to help running `docker-compose` commands.

Source `.env.sh` in every shell where you want to run the utils found in `bin`
or if you want to run `docker` manually (as opposed to from `make`).


## Operation

See `Makefile` for all supported operations.

Use `make build` to build images, then use `make up-background`.

Docker will restart the services on boot.

Docker Compose will now start all the services for you:

```shell
Starting 100conv_web_1        ... done
Starting 100conv_db_1         ... done
Starting 100conv_wordpress_1  ... done
```

🚀 Open [https://100conversations.local](https://100conversations.local) in your browser

### Running composer, mysql, php, and bash

To run normal utilities and applications within Docker containers, use the
scripts in `bin`. To add this directory to the front of your path, make
sure you source `.env.sh`.

From then on, `composer` will run `bin/composer`, which runs composer in
the PHP Docker container. There are other mappings to important binaries
provided in `bin.`


#### Use WP-CLI

```shell
docker exec -it 100conv_wordpress_1 bash
```

Login to the container

```shell
wp search-replace https://olddomain.com https://newdomain.com --allow-root
```

Run a wp-cli command

> You can use this command first after you've installed WordPress using Composer as the example above.

### Update Bedrock environment

To modify Bedrock environment configuration; you're able to edit this file `./bedrock/config/environments/development.php` (for example to use it in with `WP_ENV=development`)

```shell
Config::define('WP_DEBUG', true);
Config::define('WP_DEBUG_DISPLAY', true);
```

### Useful Docker Commands

When making changes to the Dockerfile, use:

```shell
docker-compose up -d --force-recreate --build
```

Login to the docker container

```shell
docker exec -it 100conv_wordpress_1 bash
```

Stop

```shell
docker-compose stop
```

Down (stop and remove)

```shell
docker-compose down --remove-orphans
```

Cleanup

```shell
docker-compose rm -v
```

Recreate

```shell
docker-compose up -d --force-recreate
```
