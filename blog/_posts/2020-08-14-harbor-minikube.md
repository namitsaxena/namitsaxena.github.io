---
layout: page
title: Running Harbor container registry on Kubernetes
categories: [tech]
tags: [tech, kubernetes, harbor]
comments: true
---

## Overview
[Harbor](https://goharbor.io/) is relatively new and popular container registry. If you are running your own kubernetes/container environment, then using this as a private registry may be a good choice. This covers the basic deployment process using minikube.

## Prerequisites
* kubernetes setup including, minikube and kubectl
* helm 3
* virtualbox

## Installation
* start minikube using virtualbox driver, and enable ingress
  ```
  minikube start --vm-driver virtualbox
  minikube addons enable ingress
  # to see the list of add ons
  minikube addons list
  ```
  * Because we will be setting up harbor services using ingress, we need the virtualbox driver
  * If we were using NodePort, we could have used docker driver
* Clone [harbor repo](https://github.com/goharbor/harbor-helm), and cd to the repo
  * we are deploying staright from repo 
* Setup harbor namespace
```
kubectl create namespace harbor
kubectl config set-context --current --namespace harbor
helm install harbor .
```
* Setup hosts file (simulate dns for ingress)
  * find out the ip address for ingress
  
  ```
  | => minikube ip
  192.168.99.103

  | => kg ingress
  eNAME                           CLASS    HOSTS                  ADDRESS   PORTS     AGE
  harbor-harbor-ingress          <none>   core.harbor.domain               80, 443   6m17s
  harbor-harbor-ingress-notary   <none>   notary.harbor.domain             80, 443   6m17s

  | => kubectl describe ingress 
  Name:             harbor-harbor-ingress
  Namespace:        harbor
  Address:          192.168.99.103

  ```
  
  * Setup hosts file: on mac, add to /etc/hosts, the following
  ```
  192.168.99.103 core.harbor.domain
  192.168.99.103 notary.harbor.domain
  ```
  
  * At this point you should be able to access harbor on: [https://core.harbor.domain](https://core.harbor.domain/) using the deafult username and password
    * username: admin
    * password: Harbor12345

## Configuring Docker Client
Even though harbor UI is up, docker client will not be able to access it since we haven't installed harbor certificates yet.

* Configure docker daemon to use: we can use mac's docker daemon or the one in minikube. Here we will use the one in minikube (it's closer to how linux is setup)
```
eval $(minikube docker-env)
```

* Get the certificates from kubernetes secrets
```
# the base64 command on linux would use -d 
kubectl -n harbor get secrets harbor-harbor-ingress -o jsonpath="{.data['ca\.crt']}" | base64 -D > harbor-ca.crt
```

* Install certificates in docker daemon:

  ```
  # copy the certificate
  scp -o IdentitiesOnly=yes -i $(minikube ssh-key) harbor-ca.crt docker@$(minikube ip):./harbor-ca.crt

  # install the certificate by logging in)
  minikube ssh
  sudo mkdir -p /etc/docker/certs.d/core.harbor.domain
  sudo cp harbor-ca.crt /etc/docker/certs.d/core.harbor.domain
  
  # exit back to mac
  exit
  
  ```
* configure notary certificates
  Same ingress is used by the notary/Docker Content Trust as well, and the certificate needs to be configured for it's use as well (more on this below). 
  We will be configuring this on the docker client side (ie mac)
  ```
  | => cp harbor-ca.crt /Users/admin/.docker/tls/notary.harbor.domain
  ```

* Now test the client by logging in, and pushing an image
  ```
  docker login core.harbor.domain --username=admin --password Harbor12345
  docker pull hello-world
  docker tag hello-world core.harbor.domain/library/hello-world:1
  docker push core.harbor.domain/library/hello-world:1

  ```
