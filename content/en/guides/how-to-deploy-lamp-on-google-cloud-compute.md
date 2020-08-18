---
title: "How to Deploy LAMP servers on Google Cloud Compute using GCloud"
date: 2020-08-17T21:27:59-04:00
draft: false
author: serainville
tags:
    - Google Cloud
    - LAMP
description: |
    Learn how to deploy a LAMP server on Google Cloud Compute using the gcloud CLI and startup scripts
---

A LAMP server (Linux, Apache, MySQL, PHP) is one of the most common server setups on the Internet, even after more than a decade. It is still true that PHP powers today's web communities, despite challenges from more recent languages and frameworks. 

## Provision a Compute Instance
### Create network
### Create new instance
When creating a new compute instance you will need to give it a name, as well as specify the machine type and image to be used. These are the two most important decisions you will make when provisioning a new instance for running a LAMP server.

**Machine Type** defines how much CPU and memory a compute instance will be given. Once set these values cannot be changed, so it is imperative that you understand the basic requirements of your LAMP server prior to deploying a new instance. The machine type will determine whether the compute and memory is shared or not, with the latter being part of a more expensive tier.

**Image** is the operating system image used to build your compute instance. Most popular is an image based on Ubuntu, though you will likely find any flavour of Linux you prefer.

**Region** is where the compute instance will be hosted. 

To view a list of available machine types you can use the `gcloud compute machine-types list` command. The list of machine types is fairly comprehensive. You will likely on use this as a quick reference for the name and region.



```shell
gcloud compute instances create INSTANCE_NAMES \
    --machine-type=<MACHINE-TYPE> \
    --image-family=<OS-IMAGE-FAMILY> \
    --image-project=<OS-IMAGE-PROJECT> \
    --subnet=<SUBNET-NAME> \
    --boot-disk-size=<SIZE|MB,GB,TB> \
    --boot-disk-type=<BOOT-DISK-TYPE> \
    --region=<GCP REGION>
```

For example, to deploy a new compute instance for a WordPress blog named `wordpress-lamp-1` on `Ubuntu 20.04` with a boot disk size of 10GB in us-central-a, we would execute the following command.

```shell
gcloud compute instances create wordpress-lamp-1 --machine-type=f1-micro --image-family=ubuntu-2004-lts --image-project=ubuntu-os-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --subnet=default  --zone=us-central1-a
```
    | Output
```shell
Created [https://www.googleapis.com/compute/v1/projects/serverlab/zones/us-central1-a/instances/wordpress-lamp-1].
NAME              ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
wordpress-lamp-1  us-central1-a  f1-micro                   10.128.0.32  35.223.143.236  RUNNING
```

### Startup Scripts
To beauty of running your services in the cloud is the ease of being able to provision and teardown instances. Scaling on demand is also a possibility to aid you when you have a surge of traffic or work. Having to manually config a newly provisioned compute instance, however, is not an enjoyable exersice, and down right blocks auto scaling.

Startup scripts allow us to execute commands during a compute instance's provisioning step. 

Create a new file named `startup.sh` and populate it with the command necessary to configure a basic LAMP server.

```shell
#! /bin/bash
# Update installed packages and install LAMP packages
sudo apt update
sudo apt upgrade -y
sudo apt install -y php php-mysql php-gd php-xml mysql-server apache2

# Download and install WordPress
wget https://wordpress.org/latest.tar.gz
tar xvf latest.tar.gz -C /var/www/html
```

With the startup script, create a new compute instance and provision the instance as a LAMP server. To instruct `gcloud` to use the startup script as part of the instance provisioning process, we use the `--metadata-from-file` flag along with `startup-script=<path/to/file>`. 

{{< note >}}
Local startup scripts are limited to 256 KB. If your startup scripts exceeds this limit, you will need to store the script in Cloud Storage, and specify the script URL. To use a remote file rather than a local file, use `startup-script-url` instead of `startup-script`.
{{< /note >}}

For example, to use the `startup.sh` on your local machine as the startup script, you would set the flag as `--metadata-from-file startup-script=startup.sh`. 

```shell
gcloud compute instances create wordpress-lamp-2 --machine-type=f1-micro --image-family=ubuntu-2004-lts --image-project=ubuntu-os-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --subnet=default  --zone=us-central1-a --metadata-from-file startup-script=startup.sh
```
    | Output
```shell
Created [https://www.googleapis.com/compute/v1/projects/serverlab/zones/us-central1-a/instances/wordpress-lamp-2].
NAME              ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP     STATUS
wordpress-lamp-2  us-central1-a  f1-micro                   10.128.0.33  35.239.218.206  RUNNING
```

Actions performed by the startup script will be logged to syslog. To audit and troubleshoot your startup script you can `ssh` into the instance or view the logs in the Google Compute console. 

The following is an example of startup-script log entries.

```text
Aug 18 02:15:41 wordpress-lamp-2 systemd[1]: Starting Google Compute Engine Startup Scripts...
         Starting [0;1;39mGoogle Compute Engine Startup Scripts[0m...
Aug 18 02:15:42 wordpress-lamp-2 startup-script: INFO Starting startup scripts.
Aug 18 02:15:42 wordpress-lamp-2 startup-script: INFO Found startup-script in metadata.
Aug 18 02:15:42 wordpress-lamp-2 startup-script: INFO startup-script: WARNING: apt does not have a stable CLI interface. Use with caution in scripts.
Aug 18 02:15:42 wordpress-lamp-2 systemd-resolved[415]: Server returned error NXDOMAIN, mitigating potential DNS violation DVE-2018-0001, retrying transaction with reduced feature level UDP.
Aug 18 02:15:42 wordpress-lamp-2 systemd-resolved[415]: Server returned error NXDOMAIN, mitigating potential DNS violation DVE-2018-0001, retrying transaction with reduced feature level UDP.
Aug 18 02:15:42 wordpress-lamp-2 startup-script: INFO startup-script: Hit:1 http://us-central1.gce.archive.ubuntu.com/ubuntu focal InRelease
Aug 18 02:15:42 wordpress-lamp-2 startup-script: INFO startup-script: Get:2 http://us-central1.gce.archive.ubuntu.com/ubuntu focal-updates InRelease [111 kB]
Aug 18 02:15:42 wordpress-lamp-2 startup-script: INFO startup-script: Get:3 http://us-central1.gce.archive.ubuntu.com/ubuntu focal-backports InRelease [98.3 kB]
Aug 18 02:15:42 wordpress-lamp-2 startup-script: INFO startup-script: Get:4 http://archive.canonical.com/ubuntu focal InRelease [12.1 kB]
Aug 18 02:15:42 wordpress-lamp-2 startup-script: INFO startup-script: Get:5 http://us-central1.gce.archive.ubuntu.com/ubuntu focal/universe amd64 Packages [8628 kB]
```

