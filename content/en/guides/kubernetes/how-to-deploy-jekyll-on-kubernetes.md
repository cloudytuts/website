---
title: "How to Deploy Jekyll on Kubernetes"
date: 2020-09-15T12:03:19-04:00
draft: false
author: serainville
tags:
    - kubernetes
    - jekyll
    - python
    - docker
repo: https://github.com/cloudytuts/kubernetes-in-action
description: |
    Learn how to use Docker to containerize Jekyll static-content websites and deploy them on Kubernetes clusters.
---

Websites with mostly static content should not have the costly overhead of running and maintaining a CMS and database backend. Rather, they benefit greatly by using a static content generator, such as Jekyll, to generate the HTML, CSS, and JavaScript files to be served to visitors. In this guide you will learn how to containerize static content generated websites from Jekyll and deploy them on Kubernetes.

## Jekyll Build
Jekyll is a very popular ruby-based static-content generator. The most common place to find Jekyll generated websites is on Github, which can be used to serve static websites.

Jekyll provides a templating system for designing and organizing your website's content and pages. As such, we ensure all generated pages have the same theme and a consistent user experience. 

Content is written as Markdown down files, which is another popular file format used to format text and is used extensively in Github and other version control systems. It's simplistic markdown is very human-readable and is reminicent of the old wordprocessor days from the 80's and 90's. 

### Docker Build
The first step to running your static site in Kubernetes is to build a container image of it. In this example, we will build a Docker image for our Jekyll generated website that is served by NGINX.

The Docker build process will run two stages. Docker build stages are a fantastic way of isolating your build depencies from our final artifacts, ensuring the smallest application footprint possible, as well as improves security since sensitive information is not stored. 

The first stage will use an official Ruby Docker image as its base, which be used to install all of our Jekyll build dependencies. Our Jekyll files will then be copied into the build stage and jekyll will be ran to generate our static files.

The second stage is based on an official NGINX docker image. Our Jekyll build artifacts (the HTML, JS, CSS, and other static content) will be copied into the default NGINX root directory `/usr/share/nginx/html`. 

1. Create a new file at the root of your project named `Dockerfile`.
1. Add the following contents to it.
    ```dockerfile
    FROM ruby:2.7.1-buster AS build
    COPY . /app
    WORKDIR /app
    RUN bundle install \
        && jekyll build

    FROM nginx:1.19.2-alpine AS final
    COPY --from=build /app/_site /usr/share/nginx/html
    ```
1. Save your changes and exit the text editor.
1. Run the `docker build` command to build your new image. The `-t` flag sets a tag for the final image, which will be used to give a name and version of our image `jekyll-app:1.0.0`. The build process will likely take a few minutes, as each dependency in the Gemfile must be downloaded and compiled where necessary.
    ```shell
    docker build -t jekyll-app:1.0.0 .
    ```
    ```shell
    Sending build context to Docker daemon  119.8kB
    Step 1/6 : FROM ruby:2.7.1-buster AS build
    ---> d8ca85855516
    Step 2/6 : COPY . /app
    ---> df0e0c73dccf
    Step 3/6 : WORKDIR /app
    ---> Running in 4dc3a2c7ef22
    Removing intermediate container 4dc3a2c7ef22
    ---> 60f568f08d85
    Step 4/6 : RUN bundle install     && jekyll build
    ---> Running in e78b59e94f5f
    The dependency tzinfo (~> 1.2) will be unused by any of the platforms Bundler is installing for. Bundler is installing for ruby but the dependency is only for x86-mingw32, x64-mingw32, x86-mswin32, java. To add those platforms to the bundle, run `bundle lock --add-platform x86-mingw32 x64-mingw32 x86-mswin32 java`.
    The dependency tzinfo-data (>= 0) will be unused by any of the platforms Bundler is installing for. Bundler is installing for ruby but the dependency is only for x86-mingw32, x64-mingw32, x86-mswin32, java. To add those platforms to the bundle, run `bundle lock --add-platform x86-mingw32 x64-mingw32 x86-mswin32 java`.
    The dependency wdm (~> 0.1.1) will be unused by any of the platforms Bundler is installing for. Bundler is installing for ruby but the dependency is only for x86-mingw32, x64-mingw32, x86-mswin32. To add those platforms to the bundle, run `bundle lock --add-platform x86-mingw32 x64-mingw32 x86-mswin32`.
    Fetching gem metadata from https://rubygems.org/.........
    Fetching public_suffix 4.0.6
    Installing public_suffix 4.0.6
    Fetching addressable 2.7.0
    Installing addressable 2.7.0
    Using bundler 2.1.4
    Fetching colorator 1.1.0
    Installing colorator 1.1.0
    Fetching concurrent-ruby 1.1.7
    Installing concurrent-ruby 1.1.7
    Fetching eventmachine 1.2.7
    Installing eventmachine 1.2.7 with native extensions
    Fetching http_parser.rb 0.6.0
    Installing http_parser.rb 0.6.0 with native extensions
    Fetching em-websocket 0.5.1
    Installing em-websocket 0.5.1
    Fetching ffi 1.13.1
    Installing ffi 1.13.1 with native extensions
    Fetching forwardable-extended 2.6.0
    Installing forwardable-extended 2.6.0
    Fetching i18n 1.8.5
    Installing i18n 1.8.5
    Fetching sassc 2.4.0
    Installing sassc 2.4.0 with native extensions
    Fetching jekyll-sass-converter 2.1.0
    Installing jekyll-sass-converter 2.1.0
    Fetching rb-fsevent 0.10.4
    Installing rb-fsevent 0.10.4
    Fetching rb-inotify 0.10.1
    Installing rb-inotify 0.10.1
    Fetching listen 3.2.1
    Installing listen 3.2.1
    Fetching jekyll-watch 2.2.1
    Installing jekyll-watch 2.2.1
    Fetching rexml 3.2.4
    Installing rexml 3.2.4
    Fetching kramdown 2.3.0
    Installing kramdown 2.3.0
    Fetching kramdown-parser-gfm 1.1.0
    Installing kramdown-parser-gfm 1.1.0
    Fetching liquid 4.0.3
    Installing liquid 4.0.3
    Fetching mercenary 0.4.0
    Installing mercenary 0.4.0
    Fetching pathutil 0.16.2
    Installing pathutil 0.16.2
    Fetching rouge 3.23.0
    Installing rouge 3.23.0
    Fetching safe_yaml 1.0.5
    Installing safe_yaml 1.0.5
    Fetching unicode-display_width 1.7.0
    Installing unicode-display_width 1.7.0
    Fetching terminal-table 1.8.0
    Installing terminal-table 1.8.0
    Fetching jekyll 4.1.1
    Installing jekyll 4.1.1
    Fetching jekyll-feed 0.15.0
    Installing jekyll-feed 0.15.0
    Fetching jekyll-seo-tag 2.6.1
    Installing jekyll-seo-tag 2.6.1
    Fetching minima 2.5.1
    Installing minima 2.5.1
    Bundle complete! 6 Gemfile dependencies, 31 gems now installed.
    Use `bundle info [gemname]` to see where a bundled gem is installed.
    Post-install message from i18n:

    HEADS UP! i18n 1.1 changed fallbacks to exclude default locale.
    But that may break your application.

    If you are upgrading your Rails application from an older version of Rails:

    Please check your Rails app for 'config.i18n.fallbacks = true'.
    If you're using I18n (>= 1.1.0) and Rails (< 5.2.2), this should be
    'config.i18n.fallbacks = [I18n.default_locale]'.
    If not, fallbacks will be broken in your app by I18n 1.1.x.

    If you are starting a NEW Rails application, you can ignore this notice.

    For more info see:
    https://github.com/svenfuchs/i18n/releases/tag/v1.1.0
    Configuration file: /app/_config.yml
                Source: /app
        Destination: /app/_site
    Incremental build: disabled. Enable with --incremental
        Generating... 
        Jekyll Feed: Generating feed for posts
                        done in 0.315 seconds.
    Auto-regeneration: disabled. Use --watch to enable.
    Removing intermediate container e78b59e94f5f
    ---> 6fd14085eebf
    Step 5/6 : FROM nginx:1.19.2-alpine AS final
    ---> 6f715d38cfe0
    Step 6/6 : COPY --from=build /app/_site /usr/share/nginx/html
    ---> fb44a75708cc
    Successfully built fb44a75708cc
    Successfully tagged jekyll-app:1.0.0
    ```
1. Verify the image has been created using the `docker image` command. The output shows information about the newly created image and, more importantly, it's size of 22.1MB.
    ```shell
    docker images
    ```
    ```shell
    REPOSITORY                           TAG                                              IMAGE ID            CREATED             SIZE
    jekyll-demo                          1.0.0                                            fb44a75708cc        4 hours ago         22.1MB
    ```

## Kubernetes
### Deployment

1. Create a new file named `deployment.yaml`.
1. Add the following contents to it
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: jekyll-website
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: jekyll-website
      template:
        metadata:
          labels:
            app: jekyll-website  
        spec:
          containers:
          - name: website
            image: jekyll-app:1.0.0
            ports:
              - containerPort: 80
    ```
1. Apply the manifest to create the deployment.
    ```shell
    kubectl apply -f deployment.yaml
    ```
1. Verify the deployment was created successfully using `kubectl get deployment`
    ```shell
    kubectl get deployment
    ```
    ```shell
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    jekyll-website     3/3     3            3           30s
    ```

### Service
A Kubernete service are used to expose an application hosted on a Kubernetes cluster as a running service. Services can be assigned a routable IP address.

The following example will map a service onto the pods generated by the deployment created above. Any time a pod is created with labels that match the criteria set in the service below, it will be added as a backend pod to the service. 

1. Create a new file named `service.yaml`.
1. Add the following contents to it.
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: jekyll-website
    spec:
      selector:
        app: jekyll-website
      ports:
      - protocol: TCP
        port: 80
      type: Loadbalancer
    ```
1. Create the Kubernetes service by applying the manifest
    ```shell
    kubectl apply -f service.yaml
    ```
1. Verify the creation of your static-content service to ensure it
    ```shell
    kubectl get svc static-content
    ```
    ```shell
    NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
    jekyll-website   ClusterIP   10.245.94.93   <pending>     80/TCP     45s
    ```

## Configuring NGINX

## Liveliness Probes
In general, a container is considered healthy until its primary process exits. The unfortunate outcome of this is your application isn't necessarily running in a healthy state just because the container hasn't exited. 

### TCP Checks
One way of determining if NGINX is healthy is checking an open port, typically port 80 or 443. 

One downfall of a TCP check is it doesn't necessarily mean your application is running or accepting connection. It merely checks that the server is accepting connections. For example, an application may have a 503 error but the TCP port is open and listening, resulting in a false-positive healthy state.

1. Open your `deployment.yaml` into a text editor.
1. Add the lines highlighted below to add a TCP `readinessProbe` and a TCP `livelinessProbe` probe.
```yaml {hl_lines=["7-16"]}
spec:
  containers:
  - name: website
    image: jekyll-app:1.0.0
    ports:
    - containerPort: 80
    readinessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 20
```
1. Save your changes and reapply the deployment manifest.
    ```shell
    kubectl apply -f deployment.yaml
    ```

### HTTP Liveliness Probe
An endpoint health check determines an applications health by hitting a predesignated URI. If the request to the URI returns a response, such as HTTP Status 2XX, the application and, therefore, container are deemed healhy.

1. Open the `deployment.yaml` file into a text editor.
1. In the deployment manifest, add the following `livenessProbe` section to your `container`.
    ```yaml {hl_lines=["7-15"],linenostart=14}
    spec:
      containers:
      - name: website
        image: jekyll-app:1.0.0
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          httpHeaders:
          - name: Custom-Header
            value: Awesome
          initialDelaySeconds: 3
          periodSeconds: 3
    ```
1. Save your changes to `deployment.yaml` and reapply the manifest.
    ```shell
    kubectl apply -f deployment.yaml
    ```