---
title: "How to Upgrade PostgreSQL in Docker and Kubernetes"
date: 2020-08-25T22:24:34-04:00
draft: false
author: serainville
tags:
  - docker
  - kubernetes
  - postgres
description: |
    Learn how to upgrade your Postgres servers running in Docker or Kubernetes in a safe and reliable way to ensure data integrity. 
---

In this tutorial, you will learn how to safely upgrade your PostgreSQL server containers to more recent versions of the database server. These instructions apply to upgrading between major release (11.x -> 12.x) and do not apply to minor or bug releases. For minor and bug releases the base image can be upgraded without problems.

Applications like Postgres were not designed to run in a containerized world, as conventions introduced with Docker did not exist when they were originally written.

Instead, we will have to follow old, trusted conventions to safely upgrade Postgres in Docker or Kubernetes. You will need to dump your databases from the image running the older Postgres version, and then import the dump into the container running a new version of Postgres.

## 1. Deploy New Postgres Image
The first step is to deploy a new Postgress container using the updated image version. This container MUST NOT mount the same volume from the older Postgress container. It will need to mount a new volume for the database.

If you mount to a previous volume used by the older Postgres server, the new Postgres server will fail. Postgres requires the data to be migrated before it can load it.

## 2. Backup Running Container
You will need access to your Postgres container to execute the `pg_dumpall` command. To perform a one-time backup of all the databases, we are going to exec a command inside of the container and output a backup file to our local machine.

### Docker
If your Postgres server is running as a Docker container, you will execute the following command.
```shell
docker exec -it [postgres-container] -- pg_dumpall > dumpfile
```
### Kubernetes
If your Postgres server is running as a Kubernetes Pod, you will execute the following command.
```shell
kubectl exec -it [postgres-pod] -- pg_dumpall > dumpfile
```

## 3. Import Database into New Container
With the new Postgres container running with a new volume mount for the data directory, you will use the `psql` command to import the database dump file. During the import process Postgres will migrate the databases to the latest system schema.

### Docker
```shell
docker exec -it [new-postgres-container] -- psql < dumpfile 
```

### Kubernetes
```shell
kubectl exec -it [new-postgres-pod] -- psql < dumpfile>
```

## 4. Verify Import
Always verify the import completed successfully and without corruption. You may want to perform a vew tests against the data to ensure everything imported correctly before moving on.

## 5. Stop Old Container
Once you've verified that the new Postgres server is operating correctly and the imported backup is fine, you will need to stop the container or Pod of the old Postgres server. 

If you are running Kubernetes or Docker Swarm, you will need to delete the deployment of the Postgres service. Otherwise, both orchestrators will reschedule an older version of your Postgres server.

