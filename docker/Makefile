.PHONY: up up-background up-background-build stop build build-images wp-install composer-install nuke nuke-containers nuke-images nuke-db rebuild save-db restore-db

export COMPOSE_PROJECT_NAME := bedrock-docker
export PATH := $(PWD)/bin:$(PATH)

up:
	docker-compose up

up-background:
	docker-compose up -d

up-background-build:
	docker-compose up -d --force-recreate --build

# This could include composer-install and wp-install, but those require the
# composition to be up. Kept it simple.
build: build-images

build-images:
	docker-compose build

wp-install:
	cd ../bedrock ; wp core install --url=$$DOMAIN --title=$$APP_NAME --admin_user=$$WP_ADMIN_USER --admin_password=$$WP_ADMIN_PASSWORD --admin_email=$$WP_ADMIN_EMAIL

composer-install:
	cd ../bedrock ; composer install --no-dev --optimize-autoloader

# nuke: nuke-containers nuke-images nuke-db
nuke: nuke-images nuke-db

# WARNING - this could remove all containers
nuke-containers : stop
	-[ "$$(docker ps -a -q)" ] && docker rm -f -v $$(docker ps -a -q) || echo "No containers to remove"

nuke-images: stop
	-[ "$$(docker images -q --filter=reference=$(COMPOSE_PROJECT_NAME)*)" ] && docker rmi -f $$(docker images -q --filter=reference=$(COMPOSE_PROJECT_NAME)*) || echo "No images to remove"

nuke-db:
	sudo rm -fr data/db/*

rebuild: nuke build

save-db:
	docker exec $(COMPOSE_PROJECT_NAME)_db_1 sh -c 'exec mysqldump -uroot -p"$$MYSQL_ROOT_PASSWORD" $(COMPOSE_PROJECT_NAME)' > db_dump.sql

restore-db:
	docker exec -i $(COMPOSE_PROJECT_NAME)_db_1 sh -c 'exec mysql -uroot -p"$$MYSQL_ROOT_PASSWORD" $(COMPOSE_PROJECT_NAME)' < db_dump.sql

stop:
	docker-compose stop
