---
title: "How to Deploy Jenkins on Kubernetes"
date: 2020-08-24T21:16:54-04:00
draft: false
author: serainville
tags:
  - kubernetes
  - jenkins
  - cicd
description: |
    Learn how to deploy Jenkins on Kuberentes using persistent volumes for maintaining plugin installation state, services to expose Jenkins, and configMaps for configuring the server.
---

In this guide, you will learn how to deploy Jenkins on Kubernetes for the purpose of running a containerized build system. Jenkins is a free, open source build system written in Java. Likely the most common you build server you will find running in an software engineering or development environment. 

## What's Covered
* Using the official Jenkins Docker image 
* Deploying and managing Jenkins on Kubernetes
* Using Persistent Volumes to persist Jenkins state
* Exposing Jenkins as a Service
* Using Secrets and ConfigMaps to config SSL\TLS with Jenkins

## Namespace
Namespaces are an optional, but recommended, way of organizing resources in Kubernetes. In most cases you will likely want to move your build environment into its own namespace.

To create the namespace from the command-line using `kubectl create`, run the following command.
```shell
kubectl create namespace devops
```

Alternatively, a resource manifest can be written and applied to your Kubernetes cluster. This is the recommended approach, rather than creating the namespace manually using `kubectl create`. By creating and maintaining a manifest file we follow the princples of Infrastructure as Code.

Create a new file named `namespace.yaml` and add the following contents to it.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: cicd
```

To create the namespace using the Namespace manifest, run the `kubectl apply` command.
```shell
kubectl apply -f namespace.yaml
```

We can verify the namespace was created correctly using the `kubectl get ns` command.
```shell {hl_lines=[7]}
NAME              STATUS   AGE
default           Active   2d9h
docker            Active   2d9h
kube-node-lease   Active   2d9h
kube-public       Active   2d9h
kube-system       Active   2d9h
cicd              Active   13s
```


## Persistent Volume
A typical Jenkins installation has a plethora of plugins installed, as well as packages for supporting a teams development build workflows. Containers and Pods by nature are ephemeral, meaning, any changes we make to their states will be lost when they are stopped. Persistent Volumes allow us to persist our states beyond the life of a container or pod. 

To add a persistent volume to the Jenkins deployment we must create a Persistent Volume Claim for it. The claim 

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: jenkins-pv-claim
  namespace: cicd
  labels:
    app: jenkins
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Apply the manifest to create the Persistent Volume Claim resource for Jenkins.
```shell
kubectl apply -f jenkins-pvc.yaml
```

## Configuring Jenkins
The official Jenkins Docker image allows three predefined environment variables for configuring the server.
* JENKINS_HOME
* JENKINS_SLAVE_AGENT_PORT
* JENKINS_OPTS
* JAVA_OPTS

The values for the environment variables above can be set using Kubernetes ConfigMaps, which can be used to inject the environment variables into the running Jenkins Pods.

### Basic Configuration
The following ConfigMap demonstrate a very basic configration for Jenkins. 

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: jenkins
  namespace: cicd
data:
  java_opts: -Dhudson.footerURL=http://mycompany.com
  jenkins_slave_agent_prot: 8899
  executors.groovy: |
    import jenkins.model.*
    Jenkins.instance.setNumExecutors(5)
```

The `java_opts` and `jenkins_slave_agent_port` keys will be used to set the `JAVA_OPTS` and `JENKINS_SLAVE_AGENT_PORT` environment variables.

The `executors.groovy` key will be used to mount a file of the same name. The file is used to set the number of executors.

### SSL\TLS Configuration
Certificates should be used to secure client connections. However, due to the sensitive nature of private keys a ConfigMap is not an appropriate resource for storing them. Instead, use a Secret resource.

To support TLS with Jenkins we will create two resources: a ConfigMap and a Secret. The ConfigMap will store configration parameters for enabling TLS with Jenkins, and forcing all connections to HTTPS. The Secret is where the certifcate and private key will be stored.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: jenkins
  namespace: cicd
data:
  jenkins_opts: --httpPort=-1 --httpsPort=8083 --httpsCertificate=/var/lib/jenkins/cert --httpsPrivateKey=/var/lib/jenkins/pk
  executors.groovy: |
    import jenkins.model.*
    Jenkins.instance.setNumExecutors(5)
```
The `jenkins_opt` configMap key sets the location of certificate key-pair as a parameter when Jenkins executes. The certificate files, however, are stored in a Secret manifest to secure their contents.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: jenkins-tls
  namespace: jenkins
data:
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURTakNDQWpJQ0NRQ3NQa0ZldVhpdEFUQU5CZ2txaGtpRzl3MEJBUXNGQURCbk1Rc3dDUVlEVlFRR0V3SkQKUVRFUU1BNEdBMVVFQ0F3SFQyNTBZWEpwYnpFUU1BNEdBMVVFQnd3SFZHOXliMjUwYnpFVE1CRUdBMVVFQ2d3SwpRMnh2ZFdSNVZIVjBjekVmTUIwR0ExVUVBd3dXYW1WdWEybHVjeTVqYkc5MVpIbDBkWFJ6TG1OdmJUQWVGdzB5Ck1EQTRNall4TnpRek1UZGFGdzB5TVRBNE1qWXhOelF6TVRkYU1HY3hDekFKQmdOVkJBWVRBa05CTVJBd0RnWUQKVlFRSURBZFBiblJoY21sdk1SQXdEZ1lEVlFRSERBZFViM0p2Ym5Sdk1STXdFUVlEVlFRS0RBcERiRzkxWkhsVQpkWFJ6TVI4d0hRWURWUVFEREJacVpXNXJhVzV6TG1Oc2IzVmtlWFIxZEhNdVkyOXRNSUlCSWpBTkJna3Foa2lHCjl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUEzRXlNK0d4RzhUU2haVGhmOVVTMkdsM1lVdkRpRE1zZG9PalUKMDJDVUNqSitsbmdUZTJSTHBiVVk0RzYvQXUxaENsZVg0Rk4zWmZDZjdJNlFRTytyb1lpN2w3REV5NnllWnJ2Zgp1ZnU4YzJrWlUrVEhsUDM1V3B6MG1FekN5TnZWc0ZaVHlqY0ZrbmkrRld3VWNBbHFuY3hoUVUxVjZJREo0YVhkCjEzTTIzblJJU1dHVlo2L0pJUlJRNW5RcVdxMUF1YUp2ZHRWMEpkUFdFZVM2NnNKZnEycGpwbjgxc1BSWVlyaFAKU2cwWWVObjlMQzhFalc0ZS9NK3hjMHJlTmZQYk00VllUcW1wVEpsaDRwaTZLalV1N0xBazhFU3g5Q2NOTjk4NAp4N2NweW9zcERHbVRrQnNzMEo1V0NjZloyaUt3aXgvYkFPbXg3TjRMdFNKUjdPTjlRUUlEQVFBQk1BMEdDU3FHClNJYjNEUUVCQ3dVQUE0SUJBUUNtSnVvbldqSmxaMUYwQUdpaU1BYjVKQlNQa2xQbzZsRTErMGVmelJYbnJraUQKaFNiNVpCWU1GQnRQclhzY0pBanNhSEJYWnRRblRCWlA4ZDEzeWdFU2NSa2hPQVJmb0FTZHFyTFlBakpuekRnTgpGM3Rsb3pkWlJ6VG5qYTFuU2hLYTl1TmNscVdweUVZTitkSFVFaXpqKzZ3Ny9rRERMVHgwR1ZYRExXQnpKSThMCjg3Mkpxa1QrTU9vV2ZIeUlVZTc0NUtTc09GeWx6Zk44SWg3RlRYT0xBeFRhU3kxMk1wS1hBb2VWbGs3aEs2NlQKOU9XVXlTRHJVZGFmdkFBc0VDS3Mwc3ZwQ3h6bmY2V0ZBbkR6b216M3k5amdDaXFHZVJtMi9PcU1oNXN0S2pibApFeGJSV2EwTU5Vc0pjTUZLemp2aDlVZ2Y1VkRab1hLd0VaWTZadEQ4Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCk1JSUV2UUlCQURBTkJna3Foa2lHOXcwQkFRRUZBQVNDQktjd2dnU2pBZ0VBQW9JQkFRRGNUSXo0YkVieE5LRmwKT0YvMVJMWWFYZGhTOE9JTXl4Mmc2TlRUWUpRS01uNldlQk43WkV1bHRSamdicjhDN1dFS1Y1ZmdVM2RsOEovcwpqcEJBNzZ1aGlMdVhzTVRMcko1bXU5KzUrN3h6YVJsVDVNZVUvZmxhblBTWVRNTEkyOVd3VmxQS053V1NlTDRWCmJCUndDV3FkekdGQlRWWG9nTW5ocGQzWGN6YmVkRWhKWVpWbnI4a2hGRkRtZENwYXJVQzVvbTkyMVhRbDA5WVIKNUxycXdsK3JhbU9tZnpXdzlGaGl1RTlLRFJoNDJmMHNMd1NOYmg3OHo3RnpTdDQxODlzemhWaE9xYWxNbVdIaQptTG9xTlM3c3NDVHdSTEgwSncwMzN6akh0eW5LaXlrTWFaT1FHeXpRbmxZSng5bmFJckNMSDlzQTZiSHMzZ3UxCklsSHM0MzFCQWdNQkFBRUNnZ0VCQUtUamt6dzU1eHVSRmlCcUFzRFU3aXhzQTRlSkR0a2VpbzJ1MStWaXkwdWEKb2M5RUR1anpsLzl1dmpEMkUzaEFicnJMOXp5TG5MbXJVamhBT002eDFWZnh2Tjk4Q3NDYjhtL1l2VXM2bGNJWQpiMEd3NG9XdFZ4OHdqWThWSFZJejRReThnTGpCV0NWYXhJUEtRcjNjL25VZnpjZVA5L1l2dDJ0eXQ4b1VUWVJRClIxR1krQ0ZqdzVYSmZnZ2E5MEQ0Z3BRWDVLQ3RTbllOWHNhMkJ3MHFWN3NHUFpZeVhRVUl3aTQ2VC9rd0dwVzYKUjZmOU1MS2Zza3FDckJ6U3F3OUdkQmR6dUh5RjZjNzFjc1ZudlNoMS9RdlZQbzVROU5CWjlvaDVoMlBwSHZlOAp4RklUOEFtTGVUbk14OUt0VDNkZTJvZ3ZoOWNyek5IZkxmQmZqUkxmd2pVQ2dZRUE4NFYzNjRXaTQwcWpXWEpECmoybng4L0tTV25LMkx5MXE0YzJkejUwRi9GdktpbHlCN3Z6bUsxcFRCSTZ4bFJOUTArQkhKTkI3VXg3cmtRcnYKaW9XbmR1cWgwdDFmbElZU1pkM0pPTFlMUERqd0FtR28rZllsZ3h0Tk9lR1JRaHFQN0gxbllQN3RPV1I2aU5WZwoydmVqUFBEYmpZeFdja1pSMXJWWmo0OFQ3SGNDZ1lFQTU1WnpLOUNSVzRFTGYyQWs4SEk5RHZPMmVkb0ZxNVl5CmljVkJyNWF3SlNNVGxYdkhBOFVETnhoRTBiaExLZStoNFlmZ1ZabjRQQjRwVUpYZkZuTjNBY1RpWDVsemx4SGgKZnd5UmZnOTdLK2dteEVxV29USFhPeDNyOFhWTVhsbGE3SHlZVzljbXJxV2xjZU5mdEVSeTNUNEVwMFNkcWw0Qwp6bWQ2NkV0M3FnY0NnWUJ1TUJoQTg2anVtNWtxSWUrNzlyNUtHWnByWHJoY3hIbzJUZWw0Ulo2dHY0TDM5RCsrCnVhUVVQYnlPdFZwWkQvSmt6SGlraWNranBUd0YxeUxvVk8yZmV5OVowRjB0UVRVVjdyTGIvRk05SHE1TEJaR0YKK1FDa1FEaERWbk41cTdjdjFOWndKeW1EN0prZFRSK1VOTFVpSUFIWUhJWUpFeFI0eUhvTDRUdXNwUUtCZ0hMNQp6ZG90NVV5eHA1eW9oZzVlR1JSSVNRcjhCQjZwSmhRaU83ZEtMODl3TjdQYVRQY0JJOVNCbHdFcjV4MDkzSGZVCjlycHBBOFlORDJQejFGc1lIamhob0NYb1VHdnJNN0hZOG83TWJ0RmdvNGFHcFh3SCs1eGRBWnZTS1lVYUJic3QKTEpORUlPOUtTL1piOVZMUlBObThoYURwdndFclJXZG1GcTRuY1pTWEFvR0FaaUVhSVdXSXpqKzhBQk5yZHJ2Vwp0ZWN0NDdRNnVBa1FrOTFZY29LekN5M2JVbTJrcTI4YmZYb2w1YkRFbFFRU2FTa3BZZ0loQnNZL0lxZ3piWUo4Ci9vUUpQV0VObTlGdEpEdVpLMjBEU3o5d2NmdmdST1ZjenhPclY2UFZSMU9BR01zT0xLdlJCeXBJYll4Q2pCZm8KQ2oxNFFwT1lNWlVJZVk0cXc3cnpiL2c9Ci0tLS0tRU5EIFBSSVZBVEUgS0VZLS0tLS0K
  ```

The `tls.crt` value is a base64 encoded string of the certificate to be used.
The `tls.key` value is also a base64 encoded string of the certifcate key to be used.

Both keys in the Secret manifest must be mounted as files in the Jenkins Pod. Their location must match the `jenkins_opts` key value in the ConfigMap.

## Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: cicd
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins
        image: jenkins:lts
        ports:
          - containerPort: 8083
        env:
        - name: JENKINS_OPTS
          valueFrom:
            configMapKeyRef:
              name: jenkins
              key: jenkins_opts
        volumeMounts:
        - name: jenkins-storage
          mountPath: /var/lib/pgsql/data
        - name: jenkins-tls-cert
          mountPath: /var/lib/jenkins
      volumes:
      - name: jenkins-tls
        secret:
          secretName: jenkins-tls
          items:
          - key: tls.key
            path: pk
          - key: tls.crt
            path: cert
      - name: jenkins-storage
        persistentVolumeClaim:
          claimName: jenkins-pv-claim
```

## Service
Finally, to expose your Jenkins deployment a service resource is required. The followig example demonstrates how to create a service for Jenkins on Kubernetes.

```yaml
apiVers tkspk v1
kind: Service
metadata:
  name: jenkins
  namespace: cicd
spec:
  selector:
    app: jenkins
  ports:
  - protocol: TCP
    port: 8083
```

## Conclusion

In this guide, you learned how to deploy a Jenkins instance on Kubernetes. You also learned how persist Jenkins plugins, projects, jobs, and build states using a Persistent Volume. Additionally, TLS was included to show you how to secure connections to your Jenkins instance.
