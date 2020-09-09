---
title: "How to Backup and Restore MongoDB Deployment on Kubernetes"
date: 2020-09-03T04:52:05-04:00
draft: true
author: serainville
tags:
    - kubernetes
    - mongodb
description: |
    Learn how to backup and restore your MongoDB server running on Kubernetes and protect your data on a regular schedule.
---

## Manual Backup
Backups of MongoDB are done using the `mongodump` command. While we could manually shell into your running MongoDB server's pod and perform a database dump with it, that's not the best approach. An automated task is the best solution to ensure your data is protected on a regular basis.

### Backup databases with mongodump
One method of backing up your MongoDB server is to execute a command from an interactive shell using `kubectl exec`. The output of the command will be written to your local machine, with the destination path set with the `--out` flag.

The following command will backup all databases and output them to `./mongodb/backup` on your local machine.

```shell
kubectl exec -it <mongodb-pod-name> -- mongodump --out ./mongodb/backup
```

Alternatively, the `--dbpath` flag can be used to select a specific database. 
```shell
kubectl exec -it <mongodb-pod-name> -- mongodump --dbpath /data/db --out ./mongodb/backup
```


### Backup collections with mongodump
For targeting collections instead of an entire database, you use the `--collection` flag.
```shell
kubectl exec -it <mongodb-pod-name> -- mongodump --collection MYCOLLECTION --db DB_NAME --out ./mongodb/backup
```

## CronJobs
Kubernetes CronJobs are used to create regularily scheduled jobs to run on your cluster. Rather than manually backing up your MongoDB data yourself, a CronJob should be defined to automate the process for you.

The following CronJob will create a job that uses the community supported `mongo:4.4.0-bionic` image to mount the same persistent volume as your mongo server, back it up, and then tar it. 


```shell
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: mongodb-backup
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: mongodb-backup
            image: mongo:4.4.0-bionic
            args:
            - "/bin/sh"
            - "-c"
            - "/usr/bin/mongodump -u $MONGO_INITDB_ROOT_USERNAME -p $MONGO_INITDB_ROOT_PASSWORD -o /tmp/backup -h mongodb"
            - "tar cvzf mongodb-backup.tar.gz /tmp/backup"
            #- gsutil cp mongodb-backup.tar.gz gs://my-project/backups/mongodb-backup.tar.gz
            envFrom:
            - secretRef:
                name: mongodb-secret
            volumeMounts:
            - name: mongodb-persistent-storage
              mountPath: /data/db
          restartPolicy: OnFailure
          volumes:
          - name: mongodb-persistent-storage
            persistentVolumeClaim:
              claimName: mongodb-pv-claim
```
