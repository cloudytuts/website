---
title: "How to Solve Wordpress Redirects to Localhost 8080"
date: 2020-08-27T13:49:27-04:00
draft: false
author: serainville
tags:
  - docker
  - wordpress
  - docker-swarm
  - docker-compose
  - kubernetes
description: |
    Learn how to solve WordPress from redirecting to localhost:8080 when using the official Docker image in Docker Swarm, Docker Compose or Kubernetes. 
---

In this tutorial, you will learn how to solve the `localhost:8080` issue when using the official WordPress Docker image. 

Maybe you're starting a fresh WordPress site or you've moved your WordPress site onto a containerized platform, such as Kubernetes, Docker Swarm, or even Docker Compose. You've decided to use the official WordPress images to simply everything, but discover that your installation keeps redirecting to `localhost:8080`. 

You've analyzed your `docker-compose.yaml` file or your Kubernetes manifests, and have no configuration for WordPress to listen on port `:8080`. 

We'll attempt to walk you through a few steps to take, if you find yourself in this position.

## WordPress Options
WordPress sets the full domain and port number, if not port 80 (HTTP), of a new installation. The hostname is stored in the database under the `wp_options` table. A WordPress site's domain name is stored in the following options:

* **siteurl**
* **home**

The default values for both is `localhost:8080`. If you attempt to access your website from any other domain, WordPress will do an internal redirect to the host stored in these values.



## Setting Wordpress Options
The WordPress images do not make an environment variable available specifically for setting `wp_options`. However, there is an environment variable to accepts `siteurl` and `home` 

Traditionally, if we wanted to adjust these values we could set them from the `wp-config.php` file.

```php
define( 'WP_HOME', 'http://example.com' );
define( 'WP_SITEURL', 'http://example.com' );
```

These two values can be set using the `WORDPRESS_CONFIG_EXTRA` environment variable. How you use it depends on how you are running the WordPress Docker image.

### Docker
To define the `WP_HOME` and `WP_SITEURL` options from the command-line using the `-e` flag, you would do the following:

```shell
docker run --name some-wordpress -p 8080:80 -d \
-e WORDPRESS_CONFIG_EXTRA="define('WP_HOME', 'http://example.com'); define('WP_SITEURL', 'http://example.com');" \
wordpress:5.5.0-php7.2-apache
```

### Docker Compose
Add the `WORDPRESS_CONFIG_EXTRA` environment variable to your `docker-compose.yaml` file, iunder the `environment` key. 

```yaml
version: '3.1'

services:

  wordpress:
    image: wordpress:5.5.0-php7.2-apache
    restart: always
    ports:
      - 80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
      WORDPRESS_CONFIG_EXTRA: |
        define('WP_HOME', 'http://example.com');
        define('WP_SITEURL', 'http://example.com');
    volumes:
      - wordpress:/var/www/html
```

### Kubernetes
For Kubernetes, you can create a ConfigMap to store the `WORDPRESS_CONFIG_EXTRA` values, and then mount the ConfigMap to your WordPress Pod or Deployment.

The following is an example of a ConfigMap with the `WORDPRESS_CONFIG_EXTRA` variable.
```yaml {hl_lines["6-8"]}
apiVersion: v1
kind: ConfigMap
metadata:
  name: wordpress
data:
  WORDPRESS_CONFIG_EXTRA: |
    define('WP_HOME', 'http://example.com');
    define('WP_SITEURL', 'http://example.com');
```

The following Deployment manifest mounts the `wordpress` ConfigMap defined above, and then uses it to create an environment variable named `WORDPRESS_CONFIG_EXTRA` for the WordPress container to consume during at startup.
```yaml {hl_lins["22-24"]}
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
```