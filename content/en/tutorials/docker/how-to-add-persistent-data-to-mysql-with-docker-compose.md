---
title: "How to Add Persistent Data to Mysql with Docker Compose"
date: 2020-08-27T21:11:55-04:00
draft: false
author: serainville
tags:
  - docker
  - mysql
  - docker-compose
description: |
    Learn how to use volumes with Docker Compose to add persistent data for MySQL to your containerized database workloads.
repos: https://github.com/cloudytuts/docker-in-action
---
In this tutorial, you will learn how to create persistent volumes with Docker Compose and use them with MySQL. 

Containers are ephemeral by design, meaning they do not retain their state once they stop. For apps that have no state or manage their state through external services, such as Redis, this perfectly fine. However, database services like MySQL need to maintain state. Otherwise, its database would be purged every time its container is stopped.

Docker instroduced a feature named volumes to address the persistent data problem. Volumes allows us to mount a volume into a running container image and store data. Volumes persist longer than the Docker container they are mounted to, meaning when the container stops and new container takes its place the volume and its data will be available to the new container.

## MySQL Volumes
By default, MySQL stores its data files in the `/var/lib/mysql` directory. In order to persist data with a MySQL service we must mount a persistent volume into that directory. 

## Docker Volumes
Volumes are added to containers using the `-v` flag with the `docker run` command. The syntax of the a the `-v` value is:

```shell
<source>:<target>
```

The `source` value is the path on the Docker host that will be mounted in the container. The `target` value is where the volume will be mounted inside of the container.

The following is an example of mounting a volume in a MySQL Docker container. It mounts the relative path of `./mysql-data` from the host to the path `/var/lib/mysql` in the container.

```shell
docker run -p -d 3306:3306 -v ./mysql-data:/var/lib/mysql mysql:latest
```


## Docker Compose Volumes
Mounting a volume in Docker Compose is similar to doing so with Docker. Both set a `source` and a `target`. The difference between Docker and Docker compose is the latter uses a two step process. A volume must be defined outside of your services in order to be mounted in a service.

The are two major ways of accomplishing this task: a short form and a long form. The short is very similiar to regular Docker, but the long form provides more control over how a volume is mounted.

### Short Form
The simplist, carefree way of adding volumes to a server is to use the short form. 

In the example snippet below, we've defined a volume named `data-volume` and mounted it to the `mysql` service. The syntax for the shortform method of adding volumes to a container uses the following:
```text
<volume-name>:<target>
```

The `volume-name` references a volume defined in the top-level `volumes` key. The `target` is the path inside of the container the volume is mounted to.

```yaml
version: "3.8"
services:
  mysql:
    image: mysql:5.7.31
    volumes:
      - data-volume:/var/lib/mysql
volumes:
  data-volume:
```


### Long Form
The long form method of adding volumes allows you to define specific parameters for it. Let's explore these in the examples below.

In the the example snippet below, one volumes has been defined with the name `mysql-data`. Notice it's position within a `docker-compose.yaml` file. We define volumes in the top-level `volumes` key.

```yaml
version: "3.8"
services:
  mysql:
    ...

volumes:
  mysql-data:
```

With the volumes defined under the `volumes` key, we can mount them to our services with the `service` level `volumes` key.

```yaml
version: "3.8"
services:
  wordpress:
    ...
  mysql:
    image: mysql:5.7.31
    volumes:
    - type: volume
      source: mysql-data
      target: /var/lib/mysql


volumes:
  mysql-data:
```

* `type` sets the volumes mount type `volume`, `bind`, `tmpfs`, or `npipe`.
* `source` the source of the mount. This could be the path on the host for a bind mount, or the name of a volume defined in the top-level `volumes` key.
* `target` the path in the container where the volume is mounted.


## Further Reading
For more information regarding the topics covered in this tutorial, use following resources.
* [Docker Volumes](https://docs.docker.com/storage/volumes/)
* [Docker Compose Volumes](https://docs.docker.com/compose/compose-file/#volumes)