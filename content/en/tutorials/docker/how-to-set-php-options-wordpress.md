---
title: "How to Set PHP Options for Wordpress in Docker"
date: 2020-08-27T15:31:16-04:00
draft: false
author: serainville
tags:
  - wordpress
  - php
  - docker
  - kubernetes
  - docker-compose
  - docker-swarm
description: |
    Learn how to set PHP options when running WordPress in a container using Docker, Docker Swarm, Docker Compoes, and Kubernetes.
---

In this tutorial, you will learn how to set PHP options in Docker using an .htaccess for WordPress. 

Depending on how your PHP application is used, in this case WordPress, you may need to adjust some of the PHP options. Normally, this options are set in your php.ini file, but then can overwritten by an .htaccess file, which simplies our efforts.

The most common options you will adjust for Wordpress are:

* `memory_limit`
* `upload_max_size`
* `post_max_size`
* `upload_max_filesize`
* `max_execution_time`
* `max_input_time`

The defaults will usually be sufficient. However, if you find your self experiencing memory limits or upload limits are preventing you from adding content, these values should be adjusted.

{{< note >}}
PHP options should only be adjusted as necessary. It is recommended these values be left as default, unless you experience issues.
{{< /note >}}

In the examples below, we are going to create a `.ini` file to store our new PHP options. The file will then be added to the `conf.d` directory for PHP. The strategy of adding this file depends on what platform your Docker container is running on. We've included instructions for the most common: Docker, Docker Compose \ Swarm, and Kubernetes.

## Docker
Create a new file named `wordpress.ini` and use it to set your PHP options.

```ini
file_uploads = On
memory_limit = 256M
upload_max_filesize = 64M
post_max_size = 64M
max_execution_time = 300
max_input_time = 1000
```

When you start your container, mount the `wordpress.ini` as a volume inside of the container. It needs to be mounted as a file in the `/usr/local/etc/php/conf.d` directory.

```shell
docker run -d -p 8080:80 \
-v ./wordpress.ini:/usr/local/etc/php/conf.d/wordpress.ini \
-e WORDPRESS_DB_HOST="db:3306" \
-e WORDPRESS_DB_PASSWORD="P@ssw0rd2" \
wordpress:5.5.0-php7.2-apache
```

## Docker Compose \ Swarm

Create a new file named `wordpress.ini` and use it to set your PHP option values.

```ini
file_uploads = On
memory_limit = 256M
upload_max_filesize = 64M
post_max_size = 64M
max_execution_time = 300
max_input_time = 1000
```

And then mount the file as a volume in your container.

```yaml
version: '2'
services:
   wordpress:
     depends_on:
       - db
     image: wordpress:5.5.0-php7.2-apache
     ports:
       - "8080:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_PASSWORD: P@ssw0rd2
     volumes: 
       - ./wordpress.ini:/usr/local/etc/php/conf.d/wordpress.ini 
volumes:
    db_data:
```

## Kubernetes
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: wordpress-php-options
data:
  wordpress.ini: |
    file_uploads = On
    memory_limit 256M
    upload_max_filesize 64M
    post_max_size 64M
    max_execution_time 300
    max_input_time 1000
```

Apply the ConfigMap to create it in your Kubernetes cluster.
```shell
kubectl apply -f wordpress-htaccess-configmap.yaml
```

Update your deployment to mount the ConfigMap data key `wordpress-htaccess` as a file.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: wordpress
    labels:
        app: wordpress
spec:
    replicas: 1
    selector:
        matchLabels:
            app: wordpress
    template:
        metadata:
            labels:
                app: wordpress
        spec:
            containers:
                - name: wordpress
                  image: wordpress:5.5.0-php7.2-apache
                  ports:
                    - containerPort: 80
                  envFrom:
                    configMap:
                      name: wordpress
                  volumeMounts:
                  - name: wordpress-php-optons
                    mountPath: "/usr/local/php/conf.d/uploads.ini"
            volumes:
            - name: wordpress-htaccess
              configMap:
                configMapName: wordpress-php-options
                defaultMode: 0400
```