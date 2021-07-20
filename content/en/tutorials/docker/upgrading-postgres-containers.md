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
### pg_dumpall
The `pg_dumpall` utility is used for writing out **(dumping)** all of your PostgreSQL databases of a cluster. It accomplishes this by calling the `pg_dump` command for each database in a cluster, while also dumping global objects that are common to all databases, such as database roles and tablespaces.

The official PostgreSQL Docker image come bundled with all of the standard utilities, such as `pg_dumpall`, and it is what we will use in this tutorial to perform a complete backup of our database server.

In a typical PostgreSQL backup using the `pg_dumpall` command, you would use the `-f` flag to specify an output file. While, this can still be done with a containerized server the output file will be written inside of that container. An additional step would be required to copy it to your local filesystem.

Alternatively, to dump the output directly to your local filesystem you can use `>` to write a local file of the dump. The examples below will use this method over using the `-f` flag to simplify the backup process.

### Docker
If your Postgres server is running as a Docker container, you will need to execute the following command.
```shell
docker exec -it [postgres-container] /usr/bin/pg_dumpall -U [username] > dumpfile
```
For example, if you were dumping the entire database server from a container with the ID of `eddc550220ed` and a PostgreSQL user named `postgres`, you would run the following command.

```shell
docker exec -it eddc550220ed /usr/bin/pg_dumpall -U postgres > dumpfile
```

### Kubernetes
If your Postgres server is running as a Kubernetes Pod, you will execute the following command.
```shell
kubectl exec -it [postgres-pod] -- /usr/bin/pg_dumpall -U postgre > dumpfile
```

### Kubernetes Port-Forward
The Kubernetes example above executes a command inside of your PostgreSQL container, and then dumps the backup to your local filesystem using an unconventional method. A different approach that can be accomplished with Kubernetes is to `port-forward` your pod's exposed port onto your local system. 

By `port-forwarding` your Postgres pod you can run native tools locally against your database server.

```shell
kubectl port-forward svc/postgres 5432 &
```
```shell
Forwarding from 127.0.0.1:5432 -> 5432
Forwarding from [::1]:5432 -> 5432
```

With your Postgres svc port-forwarded to your local machine, you can now point `psql` to port 5432 on your local host to perform operations.

{{< note >}}
Connections to port-forwarded PostgreSQL services will require a username and password, if set. Unlike executing commands within the PostgreSQL container, which does not require a username or password by default, even if one is set.
{{< /note >}}

The example command below connects to the Postgres service running on a Kubernetes cluster with user `postgres`. The `-W` causes `pg_dumpall` to prompt for a password, and the `-f` flag sets the output file of the backup.

```shell
pg_dumpall -h 127.0.0.1 -p 5432 -U postgres -W -f database.bkp
```
```shell
Password: 
Handling connection for 5432
```

If your connection was successful, a new dump file will be found on your local filesystem.


## 3. Import PostgreSQL Dump into New Container
With the new Postgres container running with a new volume mount for the data directory, you will use the `psql` command to import the database dump file. During the import process Postgres will migrate the databases to the latest system schema.

### Docker
```shell
docker exec -it [new-postgres-container] psql < dumpfile 
```

### Kubernetes
```shell
kubectl exec -it [new-postgres-pod] -- psql < dumpfile
```


## 4. Verify Import
Always verify the import completed successfully and without corruption. You may want to perform a few tests against the data to ensure everything imported correctly before moving on.

## 5. Stop Old Container
Once you've verified that the new Postgres server is operating correctly and the imported backup is fine, you will need to stop the container or Pod of the old Postgres server. 

If you are running Kubernetes or Docker Swarm, you will need to delete the deployment of the Postgres service. Otherwise, both orchestrators will reschedule an older version of your Postgres server.

## Further Reading

* [Official documentation](https://www.postgresql.org/docs/current/app-pg-dumpall.html) for `pg_dumpall`
* [Official documentation](https://www.postgresql.org/docs/current/app-psql.html) for `psql`
* [Official PostgreSQL Docker Hub](https://hub.docker.com/_/postgres) page.
