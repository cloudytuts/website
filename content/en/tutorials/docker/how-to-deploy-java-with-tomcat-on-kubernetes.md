---
title: "How to Deploy Java Apps with Tomcat on Kubernetes"
date: 2020-09-01T22:15:54-04:00
draft: false
author: serainville
tags:
  - java
  - tomcat
  - docker
description: |
    Learn how to deploy your Java applications on Tomcat with Kubernetes, and how to configure your Tomcat install.
---


In this tutorial, you will learn how to deploy your Java Webapps on Tomcat with Docker. 

Apache provides a large list of Docker images for running Tomcat as a container. In fact, there likely more Tomcat images than most any project hosted on Docker Hub. The caveat is they all run OpenJDK. While OpenJDK has come along ways since its inception, there are still some differences between it and Java JDK.  

## Docker Build
Your first step in running your web application with Tomcat in Docker is to build an Docker image for your app. We will cover two methods for building your image: 
* Adding artifacts from a previous Java build to the Tomcat base image.
* Using Multistage builds to compile your Java Webapp and building a final, slim image based off of the Tomcat base image.

{{< note >}}
By default, no user is included in the "manager-gui" role required to operate the "/manager/html" web application. As part of your Docker image build, a `tomcat-users.xml` should be added.
{{< /note >}}

### Single Stage
A single stage Docker build is a basic build, and is the most commonly used method. With this approach it is expected that you already have all of the artifacts required, and that you are adding these artifacts to your image.

```dockerfile
FROM tomcat:10-jdk14-openjdk-slim 
COPY ./webapp.jar /usr/local/tomcat
COPY ./tomcat-users.xml /usr/local/tomcat/conf
```

### Multistage Build
A multistage build is a very handy way of compiling your application in a temporary container image, and then copying the artifacts into a final image. The benefit is all of our source files, build tools, and other dependencies do not become part of our final image, thus, minizing the final image size and security footprint.

In the example below, we are using the community supported OpenJDK image as our build base. Within this first stage of our dockerfile we are compiling our Java Webapp.

In the second stage we are using an official Tomcat image as our base. Artifacts generated during our build stage are then copied over to the final stage, and placed in Tomcat's base directory. Also as part of our Docker build, we are adding a `tomcat-users.xml` to configure users permissions.



```dockerfile
FROM openjdk:16-slim-buster AS build
COPY ./src ./src
RUN mvn package -Ddir=/build

FROM tomcat:10-jdk14-openjdk-slim AS final
COPY --from=build ./build/*.jar /usr/local/tomcat
COPY ./tomcat-users.xml /usr/local/tomcat/conf
```

To build the image, run the `docker build` command and tag your image.
```shell
docker build -t webapp:1.0.0 .
```

Test the newly created Docker image by running it using the `docker run` command.
```shell
docker run -it --rm -p 8888:8080 webapp:1.0.0
```
If the container started correctly and Tomcat also started successfully, you should be able to access it by going to the following URL in your web browser.

```text
http://localhost:8888
```

## Running Your Tomcat Container
When running a Docker container it should be started as a daemon, which essential runs it as a background service. You will also need to map a local port to the container's exposed port. By default, the Tomcat image listens on port 8080. However, in our example we will map port 80 from our local machine to the container's port 8080.

To run the container as a daemon use the `-d` flag, and to map local port 80 to the container's port 8080 use the `-p` flag.

```shell
docker run -d -p 80:8080 webapp:1.0.0 webapp-1.0.0
```

The above command will start a container named `webap-1.0.0` using our `webapp:1.0.0` image. To verify the container started successfully and is running, use the `docker ps` command.

```shell
docker ps
```

For troubleshooting issues or monitoring application logs, use the `docker logs` command.

```shell
docker logs webapp-1.0.0
```

## Configuring SSL\TLS for Tomcat
Enabling SSL\TLS on Tomcat is a two-part process that is not simply by any means, unfortunately. You can thank Java for this. 

The first step is to create a keystore that will hold your certificate files securely. Your files will be encrypted and protected by a password to prevent unauthorized access. 

The second step is configure Tomcat to use your keystore and to enable SSL\TLS. This second step is fairly messy and how you enable depends on a lot of factors.

### Creating a Keystore
#### Step 1: Create the Keystore
You will need Java installed on your local machine, whether it's the official JDK or OpenJDK. The following example creates a new keystore at the `./webapp.keystore` with an alias of `webapp`. You will be prompted to set a password for the keystore, and it is highly recommended that you do.
```shell
$JAVA_HOME/bin/keytool -genkey -alias webapp -keyalg RSA ./webapp.keystore
```

#### Step 2: Create a Certificate Signing Request (CSR)
If you do not already have a certificate and key file, you will need to generate a CSR. This CSR should then be submitted to a certificate authority of your choice, who will then supply you with a certificate file and key file to be used by your web application.

```shell
$JAVA_HOME/bin/keytool -certreq -keyalg RSA -alias [youralias] -file [yourcertificatname].csr -keystore ./webapp.keystore
```

#### Step 3: Import the Certificate files
Once you have obtained the files from your certificate authority, it is time to import them into your keystore. The first file you will import is the root certificate, which should have been provided to you.

```shell
keytool -import -alias root -keystore ./webapp.keystore] -trustcacerts -file [path/to/the/root_certificate]
```

Second, you will import your certificate to ythe keystore.

```shell
keytool -import -alias [youralias] -keystore ./webapp.keystore -file [path/to/certificate]
```

You now have a keystore with the certificate that will be used to create secure TLS\SSL connections to your web app. Next you will have to enable SSL\TLS in Tomcat.

### Configuring Tomcat
Enable TLS\SSL in Tomcat is pretty messy, as result of how Java implemented it. We're not going to cover this part in detail, since there are multiple ways of doing it depending on how you want to configure Tomcat.

The first step is to create a custom `server.xml` file, which will be used to point to your keystore and enable SSL\TLS. A good starting point is to copy the `server.xml` file from the official Tomcat image.

To copy the `server.xml` from the image to your local filesystem, use the `docker run` command and execute a command to copy the file.

```shell
docker exec -it tomcat:10-jdk14-openjdk-slim -- cp /usr/local/tomcat/conf/server.conf ./server.conf
```

You should now have a local copy of the image's default `server.xml` file. Next, follow instructions on how to configure the `server.xml` file to use your keystore, its password, and enable SSL\TLS. A [great guide has bene published by Mulesoft](https://www.mulesoft.com/tcat/tomcat-ssl).

### Adding Keystore to Docker Image
It doesn't matter where you place your keystore file, so long as Tomcat is able to access it. In our example below, we are placing the keystore in the root directory of our container. We are also copying our own custom `server.xml` file into the Docker image, since Tomcat needs to know where your keystore is located and its password.

```dockerfile
FROM tomcat:10-jdk14-openjdk-slim AS final
COPY --from=build ./build/*.jar /usr/local/tomcat
COPY ./tomcat-users.xml /usr/local/tomcat/conf
COPY ./server.xml /usr/local/tomcat/conf
COPY ./webapp.keystore /webapp.keystore
```

### Build Tomcat Image with SSL
The final step is to build your updated Docker image with the keystore and custom `server.xml` file. We're going to tag the example build with `webap:1.0.0-tls` as a way to communicate that this version is TLS enabled. However, this is an arbritrary tag that's only useful if you need to differentiate between multiple images.

```shell
docker build -t webapp:1.0.0-tls .
```


