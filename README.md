# rails-postgres-solr

## Introduction

A basic sample app with rails, postgres, and solr in docker compose.

## Technology Stack
##### Language
Ruby

##### Framework
Rails

##### Development Operations
Docker Compose

##### Database
PostgreSQL

##### Search
Solr

## Ports
Rails app
* http://localhost:3000

Solr
* http://localhost:8983

## Local Development Environment Setup Instructions

### 1: Clone the repository to a local directory
git clone git@github.com:harvard-library/ClassRequestTool.git

### 2: Create app environment variables

##### Create config file for environment variables
- Make a copy of the config example file `./env-example.txt`
- Rename the file to `.env`
- Replace placeholder values as necessary

*Note: The config file .env is specifically excluded in .gitignore and .dockerignore, since it contains credentials it should NOT ever be committed to any repository.*

### 3: Create database config

##### Create config file for the database connection
- Make a copy of the config example file `./config/database.yml.example`
- Rename the file to `database.yml`
- Replace placeholder values as necessary
- Note that this pulls in the environment variables that are set in the `.env` file

The database gets created automatically by passing in environment variables into the postgres in the docker compose file. The volume mounts the postgres data to the local filesystem `./postgresql/data:/var/lib/postgresql/data`. The first time the database is created, the `./postgresql/data` directory will appear on the local filesystem. To start with a fresh database, delete the ./postgres directory on the local filesystem and restart the contaienrs and it will be created again from scratch. Note that this will delete all data in the database. The postgres container is for development use only, when the application is deployed to the docker server it will be connected to a database server directly.

### 4: Start

##### START

This command builds all images and runs all containers specified in the docker-compose-local.yml configuration.

A local version of the docker compose file `docker-compose-local.yml` should be used for local development. The name of the local compose file must be specified in the command with the -f option. This local docker compose file points to a local version of the dockerfile `DockerfileLocal`, which is specified in the services > example_web > build > dockerfile property.

This command uses the `docker-compose-local.yml` file to build the `DockerfileLocal` image `--build` and also starts the containers `up` in background mode `-d`.

```
docker-compose -f docker-compose-local.yml up -d --build --force-recreate
```

To restart the containers later without a full rebuild, the options `--build` and `--force-recreate` can be omitted after the images are built already.

### 5: Run commands inside a container
To run commands inside a running container, execute a shell using the `exec` command. This same technique can be used to run commands in any container that is running already.

```
docker exec -it example_web bash
```

Once inside the example_web container, Rails commands can be run such as migrations.

```
rake db:migrate
```

Alternatively, to run commands inside a container that is not running already, use the docker run command or the docker compose run command.

### 6: Stop

##### STOP AND REMOVE

This command stops and removes all containers specified in the docker-compose-local.yml configuration. This command can be used in place of the 'stop' and 'rm' commands.

```
docker-compose -f docker-compose-local.yml down
```

## More information
### Volumes
A container does not have persistent storage on its own (without a volume or bind) and the data will not be saved after restarting. A volume mount is the preferred way to persist data generated and used by docker containers.

In this example, the `example_web` container mounts in the project into a home directory in the container. The volume syntax is "host_path:container_path" e.g. `.:/home/appuser`. Any files that are generated inside the container by running commands will appear on the local filesystem. Also vise versa, any changes made to files mounted from the local filesystem will appear in the container immediately.

  ```
  example_web:
    container_name: example_web
    build:
      context: ./
      dockerfile: DockerfileLocal
    env_file: .env
    volumes:
      # host_path:container_path
      - .:/home/appuser
    ports:
      - "3000:3000"
    depends_on:
      - example_db
  ```

Read [Docker storage volumes](https://docs.docker.com/storage/volumes/) for more information.
