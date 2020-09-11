---
title: "How to Update Kubernetes Deployments"
date: 2020-09-11T14:06:17-04:00
draft: false
author: serainville
tags:
    - kubernetes
    - deployments
description: |
    Learn how to update deployments and pods running in Kubernetes using a manifest and manually editing the live state.
---

In this tutorial, you will learn how to update a Kubernetes deployment using multiple methods.

In a production environment all Kubernetes resources should be created using a manifest file. 

## Updating Deployments
### Re-applying a Manifest
To most recommended method for updating and resource in Kubernetes is to modify its manifest file and re-applying it. The reason for this is: (a) a versioned history is maintained of your resource, and (b) the current state is protected in the event your cluster encounters serious problems.



```shell
kubectl apply -f my-deployment.yaml
```

### Editing live state
The second method for updating a deployment is to edit its live state. While this is an acceptable means of updating a resource in Kubernetes, it is discouraged as the state could be lost if a cluster experiences serious problems.

```shell
kubectl edit deployment my-app
```

## Output Manifest File on Cluster
In the event you do not have the original manifest file used to create a deployment, a new manifest file can be created by outputting the deployment to file using the `kubectl` command with the `-o` flag.

```shell
kubectl get deployment postgres -o yaml > postgres-deployment.yaml
```

As you can see from the example output below the YAML output includes all of the fields of a deployment resource, whether set or not, including those added by Kubernetes that should not exist in the manifest outside of the cluster. 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"postgres","namespace":"feedorus-dev"},"spec":{"replicas":1,"selector":{"matchLabels":{"app":"postgres"}},"template":{"metadata":{"labels":{"app":"postgres"}},"spec":{"containers":[{"envFrom":[{"secretRef":{"name":"postgres-secrets"}},{"configMapRef":{"name":"postgres-configmap"}}],"image":"postgres:12.4-alpine","name":"postgres","ports":[{"containerPort":5432}],"volumeMounts":[{"mountPath":"/var/lib/pgsql/data","name":"postgres-database-storage"}]}],"volumes":[{"name":"postgres-database-storage","persistentVolumeClaim":{"claimName":"postgres-pv-claim"}}]}}}}
  creationTimestamp: "2020-09-08T03:35:00Z"
  generation: 1
  managedFields:
  - apiVersion: apps/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:kubectl.kubernetes.io/last-applied-configuration: {}
      f:spec:
        f:progressDeadlineSeconds: {}
        f:replicas: {}
        f:revisionHistoryLimit: {}
        f:selector:
          f:matchLabels:
            .: {}
            f:app: {}
        f:strategy:
          f:rollingUpdate:
            .: {}
            f:maxSurge: {}
            f:maxUnavailable: {}
          f:type: {}
        f:template:
          f:metadata:
            f:labels:
              .: {}
              f:app: {}
          f:spec:
            f:containers:
              k:{"name":"postgres"}:
                .: {}
                f:envFrom: {}
                f:image: {}
                f:imagePullPolicy: {}
                f:name: {}
                f:ports:
                  .: {}
                  k:{"containerPort":5432,"protocol":"TCP"}:
                    .: {}
                    f:containerPort: {}
                    f:protocol: {}
                f:resources: {}
                f:terminationMessagePath: {}
                f:terminationMessagePolicy: {}
                f:volumeMounts:
                  .: {}
                  k:{"mountPath":"/var/lib/pgsql/data"}:
                    .: {}
                    f:mountPath: {}
                    f:name: {}
            f:dnsPolicy: {}
            f:restartPolicy: {}
            f:schedulerName: {}
            f:securityContext: {}
            f:terminationGracePeriodSeconds: {}
            f:volumes:
              .: {}
              k:{"name":"postgres-database-storage"}:
                .: {}
                f:name: {}
                f:persistentVolumeClaim:
                  .: {}
                  f:claimName: {}
    manager: kubectl
    operation: Update
    time: "2020-09-08T03:35:00Z"
  - apiVersion: apps/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          f:deployment.kubernetes.io/revision: {}
      f:status:
        f:availableReplicas: {}
        f:conditions:
          .: {}
          k:{"type":"Available"}:
            .: {}
            f:lastTransitionTime: {}
            f:lastUpdateTime: {}
            f:message: {}
            f:reason: {}
            f:status: {}
            f:type: {}
          k:{"type":"Progressing"}:
            .: {}
            f:lastTransitionTime: {}
            f:lastUpdateTime: {}
            f:message: {}
            f:reason: {}
            f:status: {}
            f:type: {}
        f:observedGeneration: {}
        f:readyReplicas: {}
        f:replicas: {}
        f:updatedReplicas: {}
    manager: kube-controller-manager
    operation: Update
    time: "2020-09-08T03:35:17Z"
  name: postgres
  namespace: feedorus-dev
  resourceVersion: "4896255"
  selfLink: /apis/apps/v1/namespaces/feedorus-dev/deployments/postgres
  uid: e8b2cf86-723d-4a1f-a677-67ad31809401
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: postgres
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: postgres
    spec:
      containers:
      - envFrom:
        - secretRef:
            name: postgres-secrets
        - configMapRef:
            name: postgres-configmap
        image: postgres:12.4-alpine
        imagePullPolicy: IfNotPresent
        name: postgres
        ports:
        - containerPort: 5432
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: /var/lib/pgsql/data
          name: postgres-database-storage
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - name: postgres-database-storage
        persistentVolumeClaim:
          claimName: postgres-pv-claim
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2020-09-08T03:35:17Z"
    lastUpdateTime: "2020-09-08T03:35:17Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2020-09-08T03:35:00Z"
    lastUpdateTime: "2020-09-08T03:35:17Z"
    message: ReplicaSet "postgres-7b7484746c" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1
```

After cleaning up the manifest file we are left with the following, which resembles the original manifest used to create the deployment.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: feedorus-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - envFrom:
        - secretRef:
            name: postgres-secrets
        - configMapRef:
            name: postgres-configmap
        image: postgres:12.4-alpine
        imagePullPolicy: IfNotPresent
        name: postgres
        ports:
        - containerPort: 5432
          protocol: TCP
        volumeMounts:
        - mountPath: /var/lib/pgsql/data
          name: postgres-database-storage
      volumes:
      - name: postgres-database-storage
        persistentVolumeClaim:
          claimName: postgres-pv-claim
```


## Conclusion
The methods above are acceptable ways of updating deployment resources, however, some are more recommended than others. 