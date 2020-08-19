---
title: "How to Create and Use Kubernetes Secrets"
date: 2020-08-12T23:07:31-04:00
draft: false
author: serainville
# tags:
description: |
    Learn how to securely store your sensitive information in Kubernetes secrets and access those secrets in your manifests.
---

Secrets are any sensitive information used by applications and services. Nearly all applications use sensitive configuration information, from database credentials to TLS\SSL certificates. For obvious security reasons, it is a good proactive to keep your code separate from your secrets.

Kubernetes Secrets allow us to store our sensitive information separately from our resources that use it. In this tutorial, you will learn how to create secrets in Kubernetes, and also how to use those secrets in your other resources, such as pods and deployments.

## Creating Secrets
To begin we will create secrets from the shell using `kubectl`. The basic syntax of the `kubectl create secret` command is the following.
```shell
kubectl create secret <type> <name> <flags>
```

There are three types of secrets that can be created from the command-line:
* docker-registry
* generic
* tls

### Generic Literal Strings
To set a secret using a string from the command-line, use the `--from-literal` flag. There is no limit to the number of `--from-literal` flags that can be used.
```shell
kubectl create secret generic example-secrets --from-literal=db.password=super-secret-password
```

### Generic From File
Alternatively, secrets can be imported from a properties file. The syntax for creating secrets from a file 
```shell
kubectl create secret generic example-secrets --from-file=<filepath>
```

For example, a properties file named `DB-credentials.txt` has bas the following content.

```text
db.password=super-secret-password
db.user=app1-prod
```

To create secrets for `db.password` and `db.user` in a secret named `example-secrets`, the following `kubectl` command would be executed.

```shell
kubectl create secret generic example-secrets --from-file=db-credentials.txt
```


## Secrets Manifests
{{< warning "Security" >}}
Storing secrets manifests in a git repository could leak sensitive information.
{{< /warning >}}
A Secrets manifest is another option for creating Kubernetes secrets. The Kubernetes API for secrets is fairly basic, where properties are defined in a `data` key. 

The example fellow adds data keys for a web application: `db.password`, `db.user`, and `certificate`. Look carefully at the values for each key. While the `db.password` and `db.user` could pass clear text values, the `certificate` value is obviously not a clear text certificate. All values under the `data` key of a **Secrets** manifest must be Base64 encoded.

```yaml
apiVersion: v1
kind: Secrets
metadata:
  name: my-secrets
data:
  db.password: c3VwZXItc2VjcmV0LXBhc3N3b3Jk
  db.user: ZGVtby1hcHAtcHJvZA==
  certificate: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUZRakNDQXlvQ0NRQ3BrQ1ZUNm9UdzhqQU5CZ2txaGtpRzl3MEJBUXNGQURCak1Rc3dDUVlEVlFRR0V3SkQKUVRFUU1BNEdBMVVFQ0F3SFQyNTBZWEpwYnpFUU1BNEdBMVVFQnd3SFZHOXliMjUwYnpFVE1CRUdBMVVFQ2d3SwpRMnh2ZFdSNVZIVjBjekViTUJrR0ExVUVBd3dTZDNkM0xtTnNiM1ZrZVhSMWRITXVZMjl0TUI0WERUSXdNRGd4Ck16QXpNVGd3TUZvWERUSXhNRGd4TXpBek1UZ3dNRm93WXpFTE1Ba0dBMVVFQmhNQ1EwRXhFREFPQmdOVkJBZ00KQjA5dWRHRnlhVzh4RURBT0JnTlZCQWNNQjFSdmNtOXVkRzh4RXpBUkJnTlZCQW9NQ2tOc2IzVmtlVlIxZEhNeApHekFaQmdOVkJBTU1FbmQzZHk1amJHOTFaSGwwZFhSekxtTnZiVENDQWlJd0RRWUpLb1pJaHZjTkFRRUJCUUFECmdnSVBBRENDQWdvQ2dnSUJBTldhZ0RGbjdHODRqMXdtWnBuTnVjQmJXM3VPdnJNUE10RTBzRmJOa1BBZWMramUKTWdpejBlU0NnWldxMXdTWXRCdVhnVDJleGh3YlhRdXJnRWpqMHo2ODlSVjZCaldYNVNtbWVmQTJIUThRY24rcQptY3czRnUzTGlCdUdGWUwrNVZsaWtXVzBxS1VUZW1hWjc5dXNNZitJTUFORUlPOTRyQmVEOHZhUFV0Wm1KVGhSClJ6ZXRTSmhMa3RINVdjUTRBdlA5M2svVzNpWVpVaDRlakNBWlljenVoTmsyVGVvV1ZsQ1RnZDlnZEFoSGpENnAKVUJQUG5ncDBXS1M1SDZ3L1hEa25oOGtkYWhkZHg4Ymd0RG1taXMwU1JIelhMdWNQSFRxUW5DVVFUUG93WFNxZwowMFdhSHdKRlJsWkJYVkV1ODBPMVNkUEM5V1RVNHZkRTc5Wm9aMWU5YmJidjVGNW9wZXNxUTZBbk1NZVlSNWYvCjM1bUNBSENaOUZEaXdzZmc5SUJNNGlDTkZTeEh6VWozNktHRlNLakdRSGhxN01PaHJhVEZ1aEFJb1lUbFRSaysKTHJGYkJFaEVndVpTQlZPVmZNb0Z5MkUwUDdTa2J2V3haOURuZnpteFJnQW5RSTJPa1NydmVUdjg2S05qb1FtdAp4Y0htUDc5MTA5MGEwR3A4RGxGcVlYRStvbWsrTFBEYU1RTFBKMm9NQ0dTRFRxbjV0SklYQ3RvQjA5ZkR0UWdzCjVCdlhrZTRWZEhaVFBYMFRQYWVHbEFJVzMyNHZORU5OclpmOFlVS1RDMFpLajBTSVFQdlVvTnMzb1FuWHREelkKQ0lWem5LT0xQcVNVS0VyZGdUTlU4dVJhVmtrTXBHSFQ3WFhoNno2eThnSmxvQjJaY0hQK1J6ald1Ukk1QWdNQgpBQUV3RFFZSktvWklodmNOQVFFTEJRQURnZ0lCQUxrZ2FuZWVGLzhXQWR5TGlrQW1vaUJHcWlSdDhjNWZ2Vk1GCnpnVTVna1FSYTlYNnU0VWE2M3RvSnQxc0kzdXZZT2d4QXplMFdKTXFvZVE4a0trTDdzMTdQNUVraFk1b3A2VlYKZjlQaVJ1QXNSV3FHT3FvKzZaaVNBTHB2YW1lYkpHTVNPcFFLSVVHYURFQ0dIVG5CSTJQakRSU2VscDhlZmIrZAp4UEdzSGhlYis4ZW1XWGkrY04rNjlrUDJDQVBuNXI5ZFpXajdwdWdpSlNtRHFkU0R2TlNPVE9ySXBzUUg2QUIrCktqdmc0OEdpd1VjZW80RXhCQjJBWE40ZTZVYmVrWUYvdkNUNU10YUZQazhZNE5QZEloc0tlTWVpbXYrMCsxeVYKNFFuRWNxSGl5Slk1UmZJWjlEblU2VnNSN1U2V2N2QVM3Vk1RdDdkeFBQbm1JajhEUWVwZmNoTXZSeCt0ZUN1aQpMeVJhampEelgwZytvZUdVMTlEeVBsUVlkc2JtMnBURjdNNThicUQ2bytmelpzY2NYaHU4cXAyMHRuUzNWUWN1CjY1TWdWYkswU2F5SXhXUkNPYnpnbHk4MmI1VytrWHFZanl5YVZOYVRsOHRDT3A2VGFLM2hoLzd6Z2FyQzMvMVIKTk5JbGV3M2dPM3J6eUtBTmZKWFdjZ1ByeGJKU3pqRklvZWJLSXZsVVR4QjcxZldxU1hzekl2TGNpUDVwbTN4KwpvSDNTQ2xZbmt0U0RPOXB6blpiT0J6Um91K0hZcHl0U25XOVVJc2lLbGpOTWJCZU1ZYjdiajM1dG8raStqOGp6CkdyeVhIQlM5L2NJeEJqaUtCNWNVbXRkcGhmU2JNWXdNby9FUWJOeWlrdm84Q2M0S3pnZjMySTkza0EvRmd3VG8KZ055YWpiOUkKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo= 
```

If you are running **OSX** or **Linux** you likely have a `base64` command available out-of-the-box. To encode a string value as base64, you can run the following command:

```shell
echo -n 'my-secret-text' | base64
```
    | output
```shell
Xktc2VjcmV0LXRleHQ=
```


## Accessing Secrets as Environment Variables

## Accessing Secrets in mounted volumes

## Base64 encoding secrets

## Using strings as secrets

