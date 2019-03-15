# docker-compose-stream-loader-kafka-mysql

## Overview

The following diagram shows the relationship of the docker containers in this docker composition.

![Image of architecture](architecture.png)

This docker formation brings up the following docker containers:

1. *[bitnami/zookeeper](https://github.com/bitnami/bitnami-docker-zookeeper)*
1. *[bitnami/kafka](https://github.com/bitnami/bitnami-docker-kafka)*
1. *[mysql](https://github.com/docker-library/mysql)*
1. *[phpmyadmin/phpmyadmin](https://github.com/phpmyadmin/docker)*
1. *[senzing/mock-data-generator](https://github.com/Senzing/mock-data-generator)*
1. *[senzing/mysql-init](https://github.com/Senzing/docker-mysql-init)*
1. *[senzing/python-mysql-base](https://github.com/Senzing/docker-python-mysql-base)*
1. *[senzing/stream-loader](https://github.com/Senzing/stream-loader)*
1. *[senzing/senzing-api-server](https://github.com/Senzing/senzing-api-server)*

### Contents

1. [Expectations](#expectations)
    1. [Space](#space)
    1. [Time](#time)
    1. [Background knowledge](#background-knowledge)
1. [Preparation](#preparation)
    1. [Clone repository](#clone-repository)
    1. [Create SENZING_DIR](#create-senzing_dir)
    1. [Prerequisite software](#prerequisite-software)
1. [Using docker-compose](#using-docker-compose)
    1. [Build docker images](#build-docker-images)
    1. [Configuration](#configuration)
    1. [Run docker formation to initialize database](#run-docker-formation-to-initialize-database)
    1. [Run docker formation to read from Kafka](#run-docker-formation-to-read-from-kafka)
1. [Cleanup](#cleanup)

## Expectations

### Space

This repository and demonstration require 7 GB free disk space.

### Time

Budget 2 hours to get the demonstration up-and-running, depending on CPU and network speeds.

### Background knowledge

This repository assumes a working knowledge of:

1. [Docker](https://github.com/Senzing/knowledge-base/blob/master/WHATIS/docker.md)
1. [Docker-compose](https://github.com/Senzing/knowledge-base/blob/master/WHATIS/docker-compose.md)

## Preparation

### Clone repository

1. Set these environment variable values:

    ```console
    export GIT_ACCOUNT=senzing
    export GIT_REPOSITORY=docker-compose-stream-loader-kafka-demo
    ```

   Then follow steps in [clone-repository](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/clone-repository.md).

1. After the repository has been cloned, be sure the following are set:

    ```console
    export GIT_ACCOUNT_DIR=~/${GIT_ACCOUNT}.git
    export GIT_REPOSITORY_DIR="${GIT_ACCOUNT_DIR}/${GIT_REPOSITORY}"
    ```

### Create SENZING_DIR

If you do not already have an `/opt/senzing` directory on your local system, visit
[HOWTO - Create SENZING_DIR](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/create-senzing-dir.md).

### Prerequisite software

The following software programs need to be installed.

#### docker

1. [Install docker](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/install-docker.md)
1. Test

    ```console
    sudo docker --version
    sudo docker run hello-world
    ```

#### docker-compose

1. [Install docker-compose](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/install-docker-compose.md)
1. Test

    ```console
    sudo docker-compose --version
    ```

## Using docker-compose

### Build docker images

1. Build docker images.

    ```console
    export BASE_IMAGE=senzing/python-mysql-base

    sudo docker build \
      --tag ${BASE_IMAGE} \
      https://github.com/senzing/docker-python-mysql-base.git

    sudo docker build \
      --tag senzing/stream-loader \
      --build-arg BASE_IMAGE=${BASE_IMAGE} \
      https://github.com/senzing/stream-loader.git

    sudo docker build --tag senzing/mock-data-generator https://github.com/senzing/mock-data-generator.git
    sudo docker build --tag senzing/mysql-init          https://github.com/senzing/docker-mysql-init.git
    ```

1. Build [senzing/senzing-api-server](https://github.com/Senzing/senzing-api-server#using-docker) docker image.

### Configuration

- **SENZING_DIR** -
  Path on the local system where
  [Senzing_API.tgz](https://s3.amazonaws.com/public-read-access/SenzingComDownloads/Senzing_API.tgz)
  has been extracted.
  See [Create SENZING_DIR](#create-senzing_dir).
  No default.
  Usually set to "/opt/senzing".
- **MYSQL_ROOT_PASSWORD** -
  The password for the the database "root" user name.
  Default: "root"
- **MYSQL_STORAGE** -
  Path on local system where the database files are stored.
  Default: "/storage/docker/senzing/docker-compose-mysql-demo"
- See [github.com/Senzing/docker-mysql](https://github.com/Senzing/docker-mysql)
  for more details on how to find values for other **MYSQL_** environment variables.

### Run docker formation to initialize database

1. Launch docker-compose formation.  Example:

    ```console
    cd ${GIT_REPOSITORY_DIR}

    export SENZING_DIR=/opt/senzing

    export MYSQL_DATABASE=G2
    export MYSQL_ROOT_PASSWORD=root
    export MYSQL_USERNAME=g2
    export MYSQL_PASSWORD=g2
    export MYSQL_STORAGE=/storage/docker/senzing/docker-compose-stream-loader-kafka-demo

    sudo docker-compose --file docker-compose-mysql-init.yaml up
    ```

1. Once docker formation is up, phpMyAdmin will be available at
   [localhost:8080](http://localhost:8080).
   You can log in with Username "root" and the Password specified in `MYSQL_ROOT_PASSWORD`.

1. The database storage will persist on the local system at ${MYSQL_STORAGE}.
   The default database storage path is `/storage/docker/senzing/docker-compose-stream-loader-kafka-demo`.

1. When the following is seen in the log:

    ```console
    senzing-mysql-init exited with code 0
    ```

    the docker formation can be brought down.

    ```console
    cd ${GIT_REPOSITORY_DIR}
    sudo docker-compose --file docker-compose-mysql-init.yaml down
    ```

### Run docker formation to read from Kafka

1. Launch docker-compose formation.

    ```console
    cd ${GIT_REPOSITORY_DIR}

    export SENZING_DIR=/opt/senzing

    export MYSQL_DATABASE=G2
    export MYSQL_ROOT_PASSWORD=root
    export MYSQL_USERNAME=g2
    export MYSQL_PASSWORD=g2
    export MYSQL_STORAGE=/storage/docker/senzing/docker-compose-stream-loader-kafka-demo

    sudo docker-compose --file docker-compose-mysql-kafka.yaml up
    ```

1. Once docker formation is up, phpMyAdmin will be available at
   [localhost:8080](http://localhost:8080).
   The records received from Kafka can be viewed in the following Senzing tables:
    1. G2 > DSRC_RECORD
    1. G2 > OBS_ENT

### Test Docker container

1. Wait for the following message in the terminal showing docker log.

    ```console
    senzing-api-server | Started Senzing REST API Server on port 8080.
    senzing-api-server |
    senzing-api-server | Server running at:
    senzing-api-server | http://0.0.0.0:8080/
    ```

1. Test Senzing REST API server.
   *Note:* port 8889 on the localhost has been mapped to port 8080 in the docker container.
   See `WEBAPP_PORT` definition.
   Example:

    ```console
    export SENZING_API_SERVICE=http://localhost:8889

    curl -X GET ${SENZING_API_SERVICE}/heartbeat
    curl -X GET ${SENZING_API_SERVICE}/license
    ```

## Cleanup

In a separate (or reusable) terminal window:

1. Use environment variable describe in "[Clone repository](#clone-repository)" and "[Configuration](#configuration)".
1. Run `docker-compose` command.

    ```console
    cd ${GIT_REPOSITORY_DIR}
    sudo docker-compose --file docker-compose-mysql-kafka.yaml down
    ```

1. Delete database storage.

    ```console
    sudo rm -rf ${MYSQL_STORAGE}
    ```

1. Delete SENZING_DIR.

    ```console
    sudo rm -rf ${SENZING_DIR}
    ```

1. Delete git repository.

    ```console
    sudo rm -rf ${GIT_REPOSITORY_DIR}
    ```
