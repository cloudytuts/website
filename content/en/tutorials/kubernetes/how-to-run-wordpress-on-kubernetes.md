---
title: "How to Run WordPress on Kubernetes"
date: 2020-08-19T22:18:46-04:00
draft: false
author: serainville
tags:
    - kubernetes
    - wordpress
    - network-policies
    - mysql
description: |
    Learn how to run WordPress and MySQL on Kubernetes in production with persistent storage, network policies, configmaps, secrets, and regular backups.
repo: https://github.com/cloudytuts/kubernetes-in-action
---

WordPress continues to be the most popular content management system used by the internet. It continues to evolve with the internet and will remain a popular choice for many more years.

Running WordPress in a stable, containerized environment requires much more than just deploying the container. There are a number of additional considerations to be made.

## What's Covered
* Deploying MySQL in Kubernetes
* Deploying WordPress in Kubernetes
* Using Ingress controllers to route web traffic
* Using CronJob to run scheduled tasks, such as backups.
* Network policies to protect network services and pods
* Persistent storage to persist state for the database and WordPress

## Getting Started
### Persistent Storage
Containers are designed to be ephemeral, which means their state is lost once the container reaches the end of its lifecycle. For a database server, this is clear not desirably, as we expect our data to continue persistenting beyond the life of our service. Otherwise, our data would be lost when a pod stops. 

### Network Policies
Network policies allow us to restrict access to our services and pods using ingress (incoming) and egress (outgoing) traffic rules. Services in a production environment should only be explicitly allowed to be accessible by the public or other services. An application that does not use a database server should not be able to connect to one, for example.

In this tutorial, we are going to implement network policies between our WordPress service and its backend database server. We are going to allow ingress traffic from WordPress to the database, and egress traffic from the database to WordPress.

### Ingress Controllers
Before a service in Kubernetes can be publicly exposed a `service` resource. The service must be either exposed via a `loadBalancer` type service or through an ingress controller, in order to support advanced service features, such as round-robin traffic to bachend pods.

For web traffic it is recommended to use Ingress Controllers. An ingress controller keeps costs down by only requiring a single cloud loadbalancer, and routing traffic to services by matching domain name and URI requests.

### CronJobs
Keeping your database and WordPress environment protected is critical in an production environment. CronJobs, which are regularly scheduled jobs that execute commands, allow us to routinely backup our data. 

Two cronjobs will be created for protecting the WordPress database and our WordPress content.

## Deploying MySQL
The following steps will show you how to deploy a single MySQL instance for your WordPress site.

### Persistent Storage
In order to support a persistent database a `PersistentVolumeClaim` needs to be defined. The following creates a writable volume claim for 10GB of storage. 

Create a new YAML filed named `wordpress-pvc.yaml` with the following contents.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

To create the `PersistentVolumeClaim` we must apply our manifest against our Kubernetes cluster using `kubectl`.
```shell
kubectl apply -f wordpress-pvc.yaml
```

### Secrets
Secrets are classified as any data that should not be publicly exposed. Certificates and passwords, for example, our common secrets required by applications. For MySQL we will need a **root** password and we most definitly do not want something as sensitive as this leaking.

There are two methods for creating secrets in Kubernetes: using the `kubectl create` command or by using a manifest file. The following example shows how to create a secret for a MySQL root password.
{{< warning >}}
**Data Exposure**: Secrets created from the command-line can be viewed in your shell's history
{{< /warning >}}

```shell
kubectl create secret wordpress-mysql --literal-string=MYSQL_ROOT_PASSWORD=
```

Alternatively, secrets can be defined in a manifest file. Secrets are defined under one of two keys in a secret manifest: `data` and `stringData`. Secrets defined under `data` must be base64 encoded, while secrets under `stringData` may be written in clear text. Either way, these values are stored base64 encoded in the etcd database and encrypted on disk.'

Strings can be easily base64 encoded on OSX and Linux using the following command.

```shell
echo -n "my-secret-text" | base64
```

Create a new manifest file named `wordpress-mysql-secrets.yaml` with the following contents. The value for `db.root.password` will be used to set our MySQL server's root password, and is base64 encoded. The value for our database name is in clear text, as it is less senstive to being exposed.

{{< warning >}}
**Data Exposure**: Secret manifests should not be stored in version control
{{< /warning >}}

```yaml
apiVersion: v1
kind: Secret
metadata:
    name: wordpress-mysql
data: 
    db.root.password: bXlzcWwuZmVlZG9ydXMtZGV2
stringData:
    db.name: wordpress
```

Create the secret by applying your manifest.

```shell
kubectl apply -f wordpress-mysql-secrets.yaml
```

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.7
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: wordpress-mysql
              key: db.root.password
        - name: MYSQL_DB_NAME
          valueFrom:
            secretKeyRef:
              name: wordpress-mysql
              key: db.name
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
```

## Deploying WordPress
### ConfigMap
### Secrets
### Persistent Storage
### Deployment

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
                  volumeMounts:
                  - name: wordpress-persistent-storage
                    mountPath: /var/www/html
                  env:
                  - name: WORDPRESS_DB_HOST
                    valueFrom:
                      secretKeyRef:
                         name: wordpress
                         key: db.host
                  - name: WORDPRESS_DB_USER
                    valueFrom:
                      secretKeyRef:
                        name: wordpress
                        key: db.user
                  - name: WORDPRESS_DB_PASSWORD
                    valueFrom:
                      secretKeyRef:
                        name: wordpress
                        key: db.password
                  - name: WORDPRESS_DB_NAME
                    valueFrom:
                      secretKeyRef:
                        name: wordpress
                        key: db.name
                  - name: WORDPRESS_TABLE_PREFIX
                    valueFrom:
                      secretKeyRef:
                        name: wordpress
                        key: db.prefix
            volumes:
            - name: wordpress-persistent-storage
              persistentVolumeClaim:
                claimName: wp-pv-claim
```

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
    name: wordpress
spec:
    selector:
        app: wordpress
    ports:
        - protocol: TCP
          port: 80
```

### Ingress Controller

### Ingress
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: wordpress
spec:
  rules:
  - host: www.myblog.com
    http:
      paths:
      - backend:
          serviceName: wordpress
          servicePort: 80
        path: /
  - host: myblog.com
    http:
      paths:
      - backend:
          serviceName: wordpress
          servicePort: 80
```

## Network Policy

## Backups
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: wordpress-mysql-backup
spec:
  schedule: "0 1 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: mysql-backup
            image: ubuntu:latest
            imagePullPolicy: IfNotPresent
            args:
              - apt-get install -y mysql-client gcloud
              - mysql -h ${DB_HOST} -u ${DB_USER} -p${DB_PASSWORD} ${DB_NAME} > ${DB_NAME}.backup.sql
              - gcloud auth activate-service-account --key-file credentials.json
              - gsutil cp *.sql gs:/my-bucket
            env:
            - name: MYSQL_DB_USER
              valueFrom:
                secretKeyRef:
                - name: wordpress-mysql-secrets
                  key: db.user
            - name: MYSQL_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                - name: wordpress-mysql-secrets
                  key: db.password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                - name: wordpress-mysql-secrets
                  key: db.name
          restartPolicy: OnFailure
```
