---
title: "How to Create Immutable Configmaps and Secrets"
date: 2020-08-27T00:16:04-04:00
draft: true
author: serainville
tags:
  - kubernetes
  - configmaps
description: |
    Learn how to create and use immutable ConfigMaps and Secrets to protect application data from being modified.
---

In this tutorial, you will learn how to create and use immutable ConfigMaps and Secrets in Kubernetes. 

Immutability has been promoted to a `beta` feature in Kubernetes `1.19`. Your cluster will need to be at the version or later in order to use the immutability feature.

## Enabling Immutablility
Immunitability was introduced as an alpha feature in version `1.18` and is disabled by default. You will need to enable the `ImmutableEphemeralVolumes` feature gate in order to use it.

The `ImmutableEphemeralVolumes` feature is enabled by default in Kubernetes `1.19`. No additionalw steps are required.

## Marking Immutable
ConfigMaps and Secrets now include a key named `immutable` that accepts a boolean value. If the key is set to `true`, the ConfigMap or Secret can not be mutated. It can only be deleted and recreated. 

```yaml {hl_lines[5]}
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp
immutable: true
data:
  api.server: https://api.myservice.com
```

{{< note >}}
By default, ConfigMaps and Secrets will not be immutable. The state was be explicitly set in order for it to be enabled.
{{< /note >}}

## Benefits
The are various scenarios where data should not be mutatable, for security and reliability reasons. This could be to enforce specific security features in an application for production environments, for example. Disabling the enforcement by mutating the parameter used to enable it could result in significant harm or damage.

In basic terms, immutable data provides the following benefits:
* Protection from accidental updates that could cause outages.
* Protection against bad actors mutating data.

An additional benefit to using immutable resources is performance. Since it is not possible to modify Secrets or ConfigMaps marked as immutable, Kubernetes does not to watch for changes to these resources. This allows you to scale the number of ConfigMaps or Secrets to an enormous amount.

## Conclusion
Kubernetes has introduced Immutable ConfigMaps and Secrets. The is advantagous to anyone who wants to protect data and configurations from unwanted changes, whether through accidental updates or bad actors targeting a cluster.