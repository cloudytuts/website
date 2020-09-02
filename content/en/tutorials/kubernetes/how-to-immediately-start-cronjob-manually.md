---
title: "How to Immediately Start Cronjob Manually"
date: 2020-09-02T17:19:42-04:00
draft: false
author: serainville
tags:
    - kubernetes
    - cronjob
description: |
    In this tutorial, you will learn how to manually start a cronjob and run it immediately using the kubectl command
---

CronJobs are jobs that are scheduled to run at regular time intervals within a Kubernetes cluster. A cronjob simply creates a new job everytime it runs, and therefore, to manually run a cronjob we create a new job. 

A CronJob resource defines how jobs will be created, from the image to be used to the command it will execute. You could manually execute a job containing all of the parameters of the CronJob, but that would be a very cumbersome way of starting the job. 

Thankfully, new Kubernetes jobs can be created using a CronJob as a template. In this way we can create a new job and the image, arguments, and everything else defined in the CronJob will be used.

## Creating a Job
To create a job from the command-line you use the `kubectl create job` command. You then provide the contaimer image to use and any commands to execute when the job starts. 

For example, to run a MySQL backup by creating a job from the command-line, you may do something like the following:
```shell
kubectl create job mysql-backup --image:mysql:5.7 -- mysqldump -u backup-user -pSuperSecretPassword myapp > myapp.sql
```

Remembering the exact arguments to run everytime you create the job is difficult enough, and looking at the command above there are some serious problems with it. The most glaring issue is your secrets are exposed in your shell's history, and another problem is there is no easy may to mount volumes to store output data.

An alternative would be to define a Job Manifest, where you can specify things like volume mounts and secrets. However, you will then encounter the challenge of keeping your Job manifest, for manually starting jobs, and your CronJob, for scheduling your job, in sync.

In most environments, even highly skilled personal with the best intentions, configuration drifts rears its ugly head and eventually your CronJob manifest and Job manifest will differ.

A better solution is to create a new job based off of an existing CronJob using kubectl.

## Manually Starting CronJobs
To manually run a CronJob as a Job you run the `kubectl create job` command. You then specify the CronJob to base the job off of using the `--from` flag. Lastly, you specify a unique name for the job.

For example, to run a job based on a CronJob named "mysql-backups", you would run the following command.
```shell
kubectl create job --from=cronjob/mysql-backups mysql-backup-manual-001
```
You can verify the job is created and executed with the `kubectl get jobs` command.
```shell
NAME                                COMPLETIONS   DURATION   AGE
mysql-backup-1598893200             0/1           2d4h       2d4h
mysql-backup-1599076800             1/1           26s        89m
mysql-backup-1599078600             1/1           28s        60m
mysql-backup-1599080400             1/1           30s        30m
mysql-backups-manual-001            1/1           10s        20s
```
In the example output show above we can see five jobs were created. Four of the jobs were created by a CronJob, while the fifth job was manually started from a CronJob.


## Conclusion
In this tutorial, you learned how to run a job manually by creating it from the command-line using kubectl. You also learned a better solution, which is create a job based off of an existing CronJob in Kubernetes.