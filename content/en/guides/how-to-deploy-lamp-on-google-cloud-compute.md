---
title: "How to Deploy LAMP servers on Google Cloud Compute using GCloud"
date: 2020-08-17T21:27:59-04:00
draft: false
author: serainville
tags:
    - Google Cloud
    - LAMP
    - WordPress
description: |
    Learn how to deploy a LAMP server on Google Cloud Compute using the gcloud CLI and startup scripts
---

A LAMP server (Linux, Apache, MySQL, PHP) is one of the most common server setups on the Internet, even after more than a decade. It is still true that PHP powers today's web communities, despite challenges from more recent languages and frameworks. 

## Getting Started
### What's Covered
* Create a network using gcloud
* Create a compute instance using gcloud
* Configure database for WordPress
* Configure WordPress

## Preparing to Provision
### Selecting a Base Image
A compute instance is a virtual machine that runs an operating system. To deploy a compute instance you will need to select which operating system it will run by selecting a base image.

A list of available images can be outputted using the `gcloud compute images list` command. An unfiltered list is very extensive, so it is recommended you filter it.

```shell
glcoud compute images list
```
    | Truncated Output
```shell
NAME                                                  PROJECT            FAMILY                            DEPRECATED  STATUS
centos-6-v20200811                                    centos-cloud       centos-6                                      READY
centos-7-v20200811                                    centos-cloud       centos-7                                      READY
centos-8-v20200811                                    centos-cloud       centos-8                                      READY
coreos-alpha-2514-1-0-v20200526                       coreos-cloud       coreos-alpha                                  READY
coreos-beta-2513-2-0-v20200526                        coreos-cloud       coreos-beta                                   READY
coreos-stable-2512-3-0-v20200526                      coreos-cloud       coreos-stable                                 READY
cos-77-12371-1072-0                                   cos-cloud          cos-77-lts                                    READY
cos-81-12871-1185-0                                   cos-cloud          cos-81-lts                                    READY
cos-beta-81-12871-117-0                               cos-cloud          cos-beta                                      READY
```

A filter is applied using the `-filter` flag, which accepts regex. To filter the list for **Ubuntu** images, you would use the following command.

```shell
glcoud compute images --filter ubuntu
```
    | Output
```shell
NAME                                  PROJECT          FAMILY                   DEPRECATED  STATUS
ubuntu-1604-xenial-v20200807          ubuntu-os-cloud  ubuntu-1604-lts                      READY
ubuntu-1804-bionic-v20200807          ubuntu-os-cloud  ubuntu-1804-lts                      READY
ubuntu-2004-focal-v20200810           ubuntu-os-cloud  ubuntu-2004-lts                      READY
ubuntu-minimal-1604-xenial-v20200807  ubuntu-os-cloud  ubuntu-minimal-1604-lts              READY
ubuntu-minimal-1804-bionic-v20200806  ubuntu-os-cloud  ubuntu-minimal-1804-lts              READY
ubuntu-minimal-2004-focal-v20200729   ubuntu-os-cloud  ubuntu-minimal-2004-lts              READY
```



### Machine Type
Machine types are hardware templates for your compute instance. These templates determine the amount of memory and CPU your compute instance will be given. Your primary costs for running a compute instance will be determined by the machine type you use; machine types with more CPU and/or memory will high a much higher operating cost.

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
## Firewall Rules
If you did not specify to allow HTTP and HTTPS traffic into your compute instance, you will need to do that now.

### Allow HTTP Access
To open HTTP traffic to your newly provisioned compute instance, run the following command.
```shell
gcloud compute firewall-rules create --network=default default-allow-http --allow=tcp:80
```
### Allow HTTPS Access
To open HTTPS traffic to your newly provisioned compute instance, run the following commnad.
```shell
gcloud compute firewall-rules create --network=default default-allow-https --allow=tcp:443
```
### Allow SSH Access
If you plan on administrating your server via SSH you will need to open up access to it, as well. 
```shell
gcloud compute firewall-rules create --network=default default-allow-ssh --allow=tcp:22
```

## SSH using GCloud
When you're server is fully provisioned you can SSH into it using the `gcloud compute ssh` command. 

The basic syntax for the command is shown below. The default user will be that of the account you have logged in with gcloud. You do not need to specify a user, unless you wish to SSH into the server using a different account.
```shell
gcloud compute ssh [USER@] INSTANCE [--zone=ZONE] 
```

For example, let's log into our `wordpress-lamp-1` compute instance.

```shell
gcloud compute ssh wordpress-lamp-1 --zone=us-central1-a
```

Alternatively, if you wanted to SSH user a different account, you would use the following example:
```shell
gcloud compute ssh operator1@wordpress-lamp-1 --zone=us-central1-a
```

## Database

So far in this guide we've covered provisioning the compute instance, which is to say we've deployed a new instance and installed all of our required packages. The next step is configuring those packages and setting up a database.

{{< note >}}
If you are not SSH'd into the server, do so now.
{{< /note >}}

### Creating a database
Console into the MySQL server instance and create a new database for WordPress.
1. Console into the database server instance
    ```shell
    mysql -u root -p
    ```
1. Create a new database for your WordPress site
    ```shell
    create database wordpress_db;
    ```

### Create Database User
No application should connect to its backend database with root credentials. An application should only be granted permissions to the databases it interacts with, and only be given the exact permissions it requires to function.

1. Create a database user for WordPress. 
    ```shell
    create user 'wordpress'@'localhost' identified by 'super-secret-password';
    ```
1. Assign the user access to the WordPress database
    ```shell
    grant select,insert,delete on '*.wordpress' to 'wordpress'@'localhost';
    ```
1. Lastly, flush the database cache to apply the new user permissions.
    ```shell
    flush;
    ```

## WordPress Config
We are nearly done! All that's left to do is configure out WordPress installation to point to our database.

1. Navigate to the root directory of your WordPress installation (the directory the contents of `latest.tar.tz` were extracted to).
    ```shell
    cd path\to\extracted\wordpress
    ```
1. Copy the `wp_config.php` to `wp_config.php`
    ```shell
    cp wp_config.php wp_config.php
    ```
1. Open `wp_config.php` into the text editor of your choice.
    ```shell
    vi wp_config.php
    ```
1. Find and modify the lines that configure WordPress' connection to a database.

## Apache vhost
The very last step is to host our WordPress site through Apache. 