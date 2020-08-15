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
* configure notary certificates (optional - required only if notary or DCT is needed)
  Same ingress is used by the notary/Docker Content Trust as well, and the certificate needs to be configured for it's use as well (more on this below). 
  We will be configuring this on the docker client side (ie mac)
  ```
  | => cp harbor-ca.crt /Users/admin/.docker/tls/notary.harbor.domain
  ```

* Now test the client by logging in, and pushing an image
  ```
  | => docker login core.harbor.domain --username=admin --password Harbor12345
  WARNING! Using --password via the CLI is insecure. Use --password-stdin.
  Login Succeeded

  docker pull hello-world
  
  | => docker tag hello-world core.harbor.domain/library/hello-world:1
  | => docker push core.harbor.domain/library/hello-world:1
  The push refers to repository [core.harbor.domain/library/hello-world]
  9c27e219663c: Layer already exists 
  1: digest: sha256:90659bf80b44ce6be8234e6ff90a1ac34acbeb826903b02cfa0da11c82cbc042 size: 525

  ```

## Using Docker Content Trust
* The certificates required for notary have already been setup (see above)
* We can enable DCT by configuring certain environment variables
  ```
  export DOCKER_CONTENT_TRUST=1
  export DOCKER_CONTENT_TRUST_SERVER=https://notary.harbor.domain
  ```
* Now we can sign and push an image. While there are other advanced ways of doing this, we will be using a simple push 
  ```
  # Running in debug mode to see more details
  # also in our case we already have root keys created and loaded
  | => docker -l=debug push core.harbor.domain/library/hello-world:1
  The push refers to repository [core.harbor.domain/library/hello-world]
  9c27e219663c: Layer already exists 
  1: digest: sha256:90659bf80b44ce6be8234e6ff90a1ac34acbeb826903b02cfa0da11c82cbc042 size: 525
  Signing and pushing trust metadata
  DEBU[0001] reading certificate directory: /Users/admin/.docker/tls/notary.harbor.domain 
  DEBU[0001] crt: /Users/admin/.docker/tls/notary.harbor.domain/core_harbor_domain_ca_root.crt 
  DEBU[0001] crt: /Users/admin/.docker/tls/notary.harbor.domain/harbor-ca.crt 
  DEBU[0001] crt: /Users/admin/.docker/tls/notary.harbor.domain/ingress.tls.crt 
  DEBU[0002] No yubikey found, using alternative key storage: no library found 
  DEBU[0002] Making dir path: /Users/admin/.docker/trust/tuf/core.harbor.domain/library/hello-world/changelist 
  DEBU[0002] received HTTP status 404 when requesting root. 
  DEBU[0002] No yubikey found, using alternative key storage: no library found 
  DEBU[0002] No yubikey found, using alternative key storage: no library found 
  Enter passphrase for root key with ID c7c1603: 
  DEBU[0009] generated ECDSA key with keyID: d95c306804340d51995c30a1dbaa81ba8ea9f2af871eef171bdcc934e3784bc7 
  DEBU[0009] generated new ecdsa key for role: targets and keyID: d95c306804340d51995c30a1dbaa81ba8ea9f2af871eef171bdcc934e3784bc7 
  Enter passphrase for new repository key with ID d95c306:
  Repeat passphrase for new repository key with ID d95c306:
  DEBU[0018] got remote timestamp ecdsa key with keyID: abd474e548219093df0ceb233ba4196a7e8d810d1643b515a68e08828c85f732 
  DEBU[0018] got remote snapshot ecdsa key with keyID: ecdd284ddd45ad506594ee4069abfc18bbb01b85aa2c06384e52a115b5514343 
  DEBU[0018] generating new snapshot...                   
  DEBU[0018] Saving changes to Trusted Collection.        
  DEBU[0018] signing root...                              
  DEBU[0018] sign called with 1/1 required keys           
  DEBU[0018] No yubikey found, using alternative key storage: no library found 
  DEBU[0018] sign called with 0/0 required keys           
  DEBU[0018] sign targets called for role targets         
  DEBU[0018] sign called with 1/1 required keys           
  DEBU[0018] No yubikey found, using alternative key storage: no library found 
  DEBU[0018] sign called with 0/0 required keys           
  Finished initializing "core.harbor.domain/library/hello-world"
  DEBU[0018] Adding target "1" with sha256 "90659bf80b44ce6be8234e6ff90a1ac34acbeb826903b02cfa0da11c82cbc042" and size 525 bytes. 
  DEBU[0018] entered ValidateRoot with dns: core.harbor.domain/library/hello-world 
  DEBU[0018] found the following root keys: [c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286] 
  DEBU[0018] found 1 valid leaf certificates for core.harbor.domain/library/hello-world: c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286 
  DEBU[0018] found 1 leaf certs, of which 1 are valid leaf certs for core.harbor.domain/library/hello-world 
  DEBU[0018] checking root against trust_pinning config for core.harbor.domain/library/hello-world 
  DEBU[0018] checking trust-pinning for cert: c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286 
  DEBU[0018]  role has key IDs: c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286 
  DEBU[0018] verifying signature for key ID: c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286 
  DEBU[0018] root validation succeeded for core.harbor.domain/library/hello-world 
  DEBU[0018] entered ValidateRoot with dns: core.harbor.domain/library/hello-world 
  DEBU[0018] found the following root keys: [c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286] 
  DEBU[0018] found 1 valid leaf certificates for core.harbor.domain/library/hello-world: c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286 
  DEBU[0018] found 1 leaf certs, of which 1 are valid leaf certs for core.harbor.domain/library/hello-world 
  DEBU[0018] checking root against trust_pinning config for core.harbor.domain/library/hello-world 
  DEBU[0018] checking trust-pinning for cert: c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286 
  DEBU[0018]  role has key IDs: c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286 
  DEBU[0018] verifying signature for key ID: c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286 
  DEBU[0018] root validation succeeded for core.harbor.domain/library/hello-world 
  DEBU[0018] received HTTP status 404 when requesting root. 
  DEBU[0018] Loading trusted collection.                  
  DEBU[0018] entered ValidateRoot with dns: core.harbor.domain/library/hello-world 
  DEBU[0018] found the following root keys: [c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286] 
  DEBU[0018] found 1 valid leaf certificates for core.harbor.domain/library/hello-world: c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286 
  DEBU[0018] found 1 leaf certs, of which 1 are valid leaf certs for core.harbor.domain/library/hello-world 
  DEBU[0018] checking root against trust_pinning config for core.harbor.domain/library/hello-world 
  DEBU[0018] checking trust-pinning for cert: c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286 
  DEBU[0018]  role has key IDs: c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286 
  DEBU[0018] verifying signature for key ID: c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286 
  DEBU[0018] root validation succeeded for core.harbor.domain/library/hello-world 
  DEBU[0018] targets role has key IDs: d95c306804340d51995c30a1dbaa81ba8ea9f2af871eef171bdcc934e3784bc7 
  DEBU[0018] verifying signature for key ID: d95c306804340d51995c30a1dbaa81ba8ea9f2af871eef171bdcc934e3784bc7 
  DEBU[0018] changelist add: 1                            
  DEBU[0018] No yubikey found, using alternative key storage: no library found 
  DEBU[0018] applied 1 change(s)                          
  DEBU[0018] sign targets called for role targets         
  DEBU[0018] sign called with 1/1 required keys           
  DEBU[0018] No yubikey found, using alternative key storage: no library found 
  DEBU[0018] sign called with 0/0 required keys           
  DEBU[0018] generating new snapshot...                   
  DEBU[0018] signing snapshot...                          
  DEBU[0018] sign called with 1/1 required keys           
  DEBU[0018] No yubikey found, using alternative key storage: no library found 
  DEBU[0018] Client does not have the key to sign snapshot. Assuming that server should sign the snapshot. 
  Successfully signed core.harbor.domain/library/hello-world:1

  ```
* We can now verify that the image has been signed (notice that the repository key is the same that was created above while pushing)
  ```
  | => docker trust inspect core.harbor.domain/library/hello-world --pretty

  Signatures for core.harbor.domain/library/hello-world

  SIGNED TAG          DIGEST                                                             SIGNERS
  1                   90659bf80b44ce6be8234e6ff90a1ac34acbeb826903b02cfa0da11c82cbc042   (Repo Admin)

  Administrative keys for core.harbor.domain/library/hello-world

    Repository Key:	d95c306804340d51995c30a1dbaa81ba8ea9f2af871eef171bdcc934e3784bc7
    Root Key:	c1b0a7a23fcc03ab9b66a8a6b804a766614461971cbc76209654c165695e1286
  ```

## Notary Client
  * We can also perform advanced notary operation using the [notary](https://github.com/theupdateframework/notary) client. 
  ### Configuring the notary client
    * Download the notary client from [the official releases page](https://github.com/theupdateframework/notary/releases). These are pre-compiled images so just download, make executable and add to the path
    * Setup the configuration file (all arguments can also be supplied on the command line but this makes it easier). The harbor certificate file is the same we created above:
    ```
    # ignore trust pinning settings for now
    # 
    | => mkdir ~/.notary directory
    | => cat ~/.notary/config.json 
    {
      "trust_dir" : "~/.docker/trust",
      "remote_server": {
      "url": "https://notary.harbor.domain",
      "root_ca": "/Users/admin/github/harbor-helm/harbor-ca.crt"
    },
    "trust_pinning": {
      "certs": {
        "docker.com/notary": ["49cf5c6404a35fa41d5a5aa2ce539dfee0d7a2176d0da488914a38603b1f4292"]
      }
    }
   }
  ```

### Using the notary client
  * This only provides some basic sample commands, for details on how to use, please see the notary documentation
  * We can verify the signatures using the notary client
    ```
    | => notary list core.harbor.domain/library/hello-world
    NAME    DIGEST                                                              SIZE (BYTES)           ROLE
     ----    ------                                                              ------------    ----
    1       90659bf80b44ce6be8234e6ff90a1ac34acbeb826903b02cfa0da11c82cbc042    525             targets
    ```
  * To initialize a new repo
    ```
    | => notary init core.harbor.domain/library/ex1
    Root key found, using: c7c1603c6f89a72bc80165c11617483fb75480f5fe54a1bf911c3d3441ba0474
    Enter passphrase for root key with ID c7c1603: 
    Enter passphrase for new targets key with ID a03454d: 
    Repeat passphrase for new targets key with ID a03454d: 
    Enter passphrase for new snapshot key with ID 6bd9be7: 
    Repeat passphrase for new snapshot key with ID 6bd9be7: 
    Enter username: admin
    Enter password: 
    ```