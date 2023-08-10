---
title: "How to Check Memory and CPU Utilization of Docker Container"
date: 2020-08-28T23:07:53-04:00
draft: false
author: serainville
tags:
  - docker
  - cpu
  - memory
description: |
    Learn how to check a Docker container's memory and CPU utilization, as well as network traffic and disk I/O to ensure everything is running fine.
---

In this tutorial, you will learn how to use the `docker` command for checking memory and CPU utilization of your running Docker containers.

Monitoring the health of your containers is crucial for a happy and reliable environment. It's very important to know if your container is hittings its head against a CPU, Memory, Network, or Block limit, which could be severely degrading it. 

## Docker Stats
The Docker command-line tool has a `stats` command the gives you a live look at your containers resource utilization. We can use this tool to gauge the CPU, Memory, Networok, and disk utilization of every running  container.

Run the `docker stats` command to display the status of your containers.

```shell
docker stats
```
```shell
CONTAINER ID        NAME                                                                                       CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
e51898f31223        lucid-lemon      0.00%               8.824MiB / 1.944GiB   0.44%               0B / 0B             0B / 0B             7
bc97fe8bcd66        funky-monkey    0.01%               12.05MiB / 1.944GiB   0.61%               0B / 0B             0B / 0B             6
17714162a329        hairy-lemon   0.15%               82.09MiB / 1.944GiB   4.12%               0B / 0B             0B / 0B             28
340bec496cf7        silly-sunshine        0.50%               20.28MiB / 1.944GiB   1.02%               0B / 0B             0B / 0B             32
```

* Memory is listed under the **MEM USAGE / LIMIT** column. This provides a snapshot of how much memory the container is utilizing and what it's memory limit is.
* CPU utilization is listed under the **CPU %** column.
* Network traffic is represented under the **NET I/O** column. It displays the outgoing and incoming traffic consumption of a container.
* Storage utilization is shown under the **BLOCK I/O** column. This show you the amount of reads and writes a container is peforming to disk.

## No Stream
By default, the `docker stats` command will display a live stream of stats. If you would prefer outputting the first stats pull results, use the `--no-stream` flag.

```shell
docker stats --no-stream
```

```shell
CONTAINER ID        NAME                                                                                       CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
e51898f31223        k8s_postgres_postgres-778965fcc7-rc2wj_default_8c8e771b-5b1a-43ee-a9ea-e4af2c1e9319_0      0.00%               8.695MiB / 1.944GiB   0.44%               0B / 0B             0B / 0B             7
bc97fe8bcd66        k8s_wordpress_wordpress-69b59478c4-5lvkk_default_1c19bdb0-2dbc-4c74-9c23-fd1f19c785bc_0    0.01%               12.05MiB / 1.944GiB   0.61%               0B / 0B             0B / 0B             6
17714162a329        k8s_mysql_wordpress-mysql-b5ddc8dd9-4xmqr_default_7694cbcb-ae3e-4bc8-a767-56c4261202f8_0   0.12%               82.09MiB / 1.944GiB   4.12%               0B / 0B             0B / 0B             28
340bec496cf7        k8s_mongodb_mongodb-5d75bdb4d7-7phxp_default_14986721-98a2-4e22-9e4a-94c569b7702e_0        0.47%               20.46MiB / 1.944GiB   1.03%               0B / 0B             0B / 0B             32
```

## All Containers
By default, `docker stats` will only output results for running containers. If you would like to output stats for all containers you can use the `-a` or `--all` flags with the command.

```shell
docker stats --no-stream -a
```

You maybe wondering why someone would want to output stats for containers that are not running. One use case is ensuring that a container is no longer running, or displaying a list of stopped containers with the running containers and their stats.

## Bitbucket Pipelines, Gitlab CI, and Github Actions
Container-based continuous integration pipelines have grown in populartiy over the last couple of years, and all major CI tool providers support it. These pipelines run containers in a non-interactive mode, which means you will not be able to use the `docker stats` command to monitor the health of your containers. In fact, it's unlikely you will be able to run any Docker commands as part of your pipeline, as the container itself will not have access to the Docker socket for security reasons.

To overcome not being able to execture `docker stats` from within a container, we can simply add the following commands to our pipeline in addition to the steps that are already there.

```shell
- while true; do echo "Memory usage in bytes:" && cat /sys/fs/cgroup/memory/memory.memsw.usage_in_bytes; sleep 2; done & 2
- while true; do date && ps aux && echo "" && sleep 2; done &
```

The commands above capture the output of the `ps aux` command, which displays the current processes running on the container, and the memory usage of the container. The output is then streamed to the pipeline logs every 2 seconds. 

The processes run in the background, so the pipeline will continue to run as normal. The output of the commands will be displayed in the pipeline logs, which you can use to monitor the health of your container.




## Conclusion
The native Docker tools provide a limited glimps into the health of your containers, but its enough to understand how each one is utilizing system resources. We determine whether a container is CPU or Memory blocked, how much network traffic is hitting or being generated by a container, and how hard it's disk storage is being hit.

In cases where we have no access to the Docker socket, we can use the `ps aux` command to display the current processes running on the container, and the memory usage of the container.

## References

- [Atlassian: Troubleshooting Bitbucket Pipelines](https://confluence.atlassian.com/bbkb/troubleshooting-bitbucket-pipelines-1141505226.html)
- [Stackoverflow](https://stackoverflow.com/questions/74892317/how-can-i-find-a-memory-leak-or-what-is-taking-up-so-much-memory-in-parceljs)