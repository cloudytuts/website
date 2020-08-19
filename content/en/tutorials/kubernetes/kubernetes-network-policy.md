---
title: How to create Kubernetes Network Policies for WordPress
author: serainville
date: "2020-08-07"
tags:
    - kubernetes
    - wordpress
    - mysql
description: |
    Learn how to craft effective network policies in Kubernetes to secure connectivity into your cluster and between your pods.
---

Out of the box pods accepts accept traffic from any source, provided traffic can be routed to it. Internally, every pod is able to communicate with any other pod in the same cluster. In today's hyper security-aware world this raises a few alarm bells. 

Services should only ever receive traffic explicitly allowed to it. To translate, while your frontend web application should be able to accept connections from any source, the database or backend system behind your frontend application should only accept traffic from the frontend, for example. There usually is no reason to allow public access or unrelated services to access the database server, which is where network policies come in handy.

In this tutorial you will learn how to craft a secure network policy for Kubernetes and apply it.

## The Basics
The two types of policies you will be defining in your network policies are **ingress** *(incoming)* and **egress** *(outgoing)*. A Kubernetes network policy can define one of each or both together using the `policyTypes` key, which is set in the `spec` of a manifest.

```yaml
spec:
  policyTypes:
  - Ingress
  - Egress
```

### Ingress Policy Rules
Within in the spec of a network policy manifest we define our ingress rules

```yaml
  ingress:
  - from:
    - ipBlock:
        cidr: <cidr address>
        except:
        - <cidr address>
    ports:
    - protocol: <TCP|UDP>
      port: <port number>
```

### Egress Policies
Egress policies have the following markup.

```yaml
  egress:
  - to:
    - ipBlock:
        cidr: <cidr address>
    ports:
    - protocol: <TCP|UDP>
      port: <port number>
```

### Network Policy
Putting everything together, we have a manifest that look similar to the example below.

```yaml
apiVersion: v1
kind: NetworkPolicy
metadata:
    name: <name>
    namespace: <namespace>
spec:
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: <cidr address>
        except:
        - <cidr address>
    ports:
    - protocol: <TCP|UDP>
      port: <port number>
  egress:
  - to:
    - ipBlock:
        cidr: <cidr address>
    ports:
    - protocol: <TCP|UDP>
      port: <port number>
```


## Network Policy Manifests

A Kubernetes manifest defines the desired state for a resource. For network policies a manifest defines a state for ingress (incoming) and egress (outgoing) traffic. The following is an example of a network policy manifest.

```yaml
apiVersion: v1
kind: NetworkPolicy
metadata:
    name: network-policy-example
    namespace: default
spec:
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

The example of sets rules ingress rules that permit incoming TCP traffic to port 6379 only from `172.17.0.0/16`, except subnet `172.17.1.0/24`. It also permits egress traffic to `10.0.0.0/24` over **TCP** port 5978. All other traffic is blocked.

The key values for a network policy are the `policyTypes`, `ingress`, and `egress`. The `policyTypes` key sets which type of traffic the network policy will apply rules against -- incoming, outgoing, or both. In the example above the policy includes rules for both ingress and egress.

Within the `spec` of the manifest, we set the desired rules for `ingress` traffic, `egress` traffic, or both. In the example above both are defined.

## Deny All Ingress Traffic

Following the recommendation that traffic to services should always be explicit, for security reasons, the following is an example of a default **deny all** rule for ingress traffic.

```yaml
apiVersion: v1
kind: NetworkPolicy
metadata:
    name: default-deny-all
    namespace: default
spec:
  policyTypes:
  - Ingress
  ingres: {}
```

## Deny All Egress Traffic
Similar to the deny all ingress traffic, by defining an empty egress policy we can effectively block all egress (outgoing) traffic.

```yaml
apiVersion: v1
kind: NetworkPolicy
metadata:
    name: default-deny-all
    namespace: default
spec:
  policyTypes:
  - Egress
  egress: {}
```

## Using Selectors
The example manifest above applies to all pods within a cluster. However, more granular control can be given by adding selectors to your network policies. The following manifest expands on the manifest shown above to permit ingress traffic only from the same namespace and for apps with the label `project: myproject`, and pods with the label `role: frontend`. 

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: wordpress-database-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myblog
    - podSelector:
        matchLabels:
          role: wordpress
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
```

## WordPress to MySQL Network Policy
Now that we have an understanding of how to define a network policy it is time to create one for our MySQL server. The following network policy permits any traffic from the `172.17.1.0/24` network access over TCP port 3306. It also only permits egress traffic to the same network.

```yaml
apiVersion: v1
kind: NetworkPolicy
metadata:
    name: wordpress-database
    namespace: default
spec:
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.1.0/24
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
        cidr: 172.17.1.0/24
    ports:
    - protocol: TCP
```

The example above is fairly loose, as it permits *any* communications from pods residing in the `172.17.1.0/24` network space. Depending on your network topology, this could be perfectly acceptable in the case where only WordPress occupies that network space.

## Selectors
Selectors allow us to further define what type of traffic is permitted. We can granularly permit traffic that match a particular label selector or namespace, for example.

In the network policy example above we've allowed an entire /24 network access to our database server. There is no delimitation
/dəˌliməˈtāSH(ə)n/
Learn to pronounce
 between our WordPress blog servers and any other type of service that may reside in that network space. With Selectors, we can permit only pods running WordPress access to our backend database.

In the example below, we've limited ingress traffic only to pods with the labels `project: myblog` and `role: wordpress`. We are also using a selector to apply the network policy onto to pods with the label `role: wordpress-db`. 

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: wordpress-database-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: wordpress-db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myblog
    - podSelector:
        matchLabels:
          role: wordpress
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
```
