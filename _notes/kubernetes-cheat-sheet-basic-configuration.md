---
layout: page
categories: [tech]
title: Kubernetes: kubectl cheat sheet - basic configuration
tags: [kubernetes, tech, cheatsheet]
---

## kubectl cheat sheet: basic configuration

* **Configuration**
    * [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
        * Change namespace: 
            * kubectl config set-context --current --namespace=test
                * if you get error like (error: you cannot specify both a context name and —current), then copy the above to a text editor and re-type double hyphens manually
                * Alternatively the following can also be used
                    * alias kcd='kubectl config set-context $(kubectl config current-context) --namespace'
        * Find current namespace - but these will not show the namespace if it’s not set explicitly (will default to ‘default’)
            * kubectl config view --minify | grep namespace
            * kubectl config view --minify --output 'jsonpath={..namespace}'
        * List and Change Context
            * kubectl config get-contexts
            * Use a different context
                * kubectl config use-context rotest (sets the current context)
            * kubectl config set-context rotest (Sets a context entry in kubeconfig; will merge fields)
* **Misc** useful items
    * changing verbosity
        *  --v 6 (--v 6 option increases the logging level enough to let you see the requests kubectl is sending to the API server.)
    * patch: useful for modifying a single property or a limited number of properties of a resource without having to edit its definition in a text editor
        * kubectl patch deployment kubia -p '{"spec": {"minReadySeconds": 10}}'
    * kubectl create / replace / apply - [difference](https://stackoverflow.com/questions/47369351/kubectl-apply-vs-kubectl-create)
        * create/replace - imperative approach - will create the resource (or throw an error if it exists)
        * apply - declarative approach - will create if needed 
    * kubectl,  --dry-run=client (Must be "none", "server", or "client")
        * client: only print the object that would be sent, without sending it
        * server: actually submits to server but is not persisted (may reject if the object exists ; more verbose)
        * none: not a dry run - actually submits
* **Help/Documentation**
    * Help: ex for ‘run’: kubectl run --help
    * kubectl explain
        * start with: kubectl api-resource, for high level resources like pod, configmap
        * Then drill down to each specific item. Ex:
            * kubectl explain pod.spec.securityContext
        * —recursive can display all sub options as well but it can display a lot if not done specifically, Also it only lists the arguments, not the description and details
            * kubectl explain pods --recursive | grep envFrom -A3 (grep 3 lines underneath)
* **kubectl querying output**
    * [JSONPath Support](https://kubernetes.io/docs/reference/kubectl/jsonpath/)
        * Ex: kg secret/sa-ro-token-h5rgr -n sa-test -o jsonpath='{.data.token}'
        * kubectl get pods -o=jsonpath='{.items[0].metadata.name}'
        * Displaying multiple fields
            * kubectl get pods -o=jsonpath="{.items[*].metadata['.name','.namespace']}"
            * kubectl get pods -o=jsonpath="{range .items[*]}{.metadata.name}{\"\t\"}{.metadata.namespace}{\"\n\"}{end}"
                * same as above but with formatting
            * kubectl get po -o jsonpath='{range .items[*]}{.metadata.name},{.spec.containers[0].image}{"\n"}{end}' 
                * Same as above: kubectl get po -o jsonpath="{range .items[*]}{.metadata.name},{.spec.containers[0].image}{'\n'}{end}” 
        * Custom columns can also take multiple jsonpaths and display them as fields
        * kubectl get pod  -o custom-columns=CONTAINER:.spec.containers[0].name,IMAGE:.spec.containers[0].image
            * kg pod busybox -o jsonpath='{.spec.containers[0].name}’ (CONTAINER column above is same as this jsonpath)
            * The json is from the pod spec - doesn’t start with .items even when displaying multiple pods
            * kg pods -o custom-columns=APP:.metadata.labels.app ( print value of label ‘app’ ; .items[*] is NOT needed)
* **Logs & Events**
    * kubectl logs mypod --previous (or -p, logs for the previous pod(which may have failed). without this it will show for the current pod)
    * kubectl logs -f pod-name [container-name]  (-f = follow/tail logs)
    * kubectl get events
* **Performance**
    * metrics api should be available/installed
        * kubectl top node 
        * kubectl top pod
* **API**
    * [The Kubernetes API | kubernetes.io](https://kubernetes.io/docs/concepts/overview/kubernetes-api/#api-groups-and-versioning)
        * To make it easier to eliminate fields or restructure resource representations, Kubernetes supports multiple API versions, each at a different API path, such as /api/v1 or /apis/rbac.authorization.k8s.io/v1alpha1.
        * [Api Groups](https://kubernetes.io/docs/reference/using-api/#api-groups): make it easier to extend the Kubernetes API. The API group is specified in a REST path and in the apiVersion field of a serialized object.
            * groups can be enabled or disabled
    * Versions:
        * List version: kubectl api-versions
        * [Kubernetes Deprecation Policy](https://kubernetes.io/docs/reference/using-api/deprecation-policy/) | Kubernetes[kubernetes.io]
        * [Understand the Kubernetes API primitives](https://thegcpgurus.com/understand-the-kubernetes-api-primitives-cka-exam-preparation-series/) – CKA Exam Preparation Series - [thegcpgurus.com]
    * List Resources
        * kubectl api-resources
    * To find valid version for an object
        * Easiest way is to use kubectl explain. To list valid versions for a resource. Ex: ReplicaSets are only supported by
            * Ex:
                * | => kubectl explain rs
                * KIND:     ReplicaSet
                * VERSION:  apps/v1
            * list of all the resource types and their latest supported version
                * for kind in `kubectl api-resources | tail +2 | awk '{ print $1 }'`; do kubectl explain $kind; done | grep -e "KIND:" -e "VERSION:"
        * Alternative way: What version an object supports can be determined using~
            * kubectl api-resources (Ex: for deployments, the api-group is apps)
            * kubectl api-versions (Ex: has 'apps' amongst others)
            * So for Deployments, we can use: apiVersion: apps/v1
                * Alternatively use 'kubectl create’ for the supported objects
                * if version not specified, use v1~~ (ex: 
    * | => kubectl cluster-info
        * Kubernetes master is running at https://127.0.0.1:32768
        * KubeDNS is running at https://127.0.0.1:32768/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
        * curl https://127.0.0.1:32768 -k (by default will give valid son reply with forbidden message
    * Starting proxy
        * kubectl proxy
            * Starting to serve on 127.0.0.1:8001
        * curl  127.0.0.1:8001 (this will list all the valid version paths. Ex: /apis/apps/v1)
    * Using kubectl to run api
        *  kubectl get --raw=/apis/apps/v1/deployments