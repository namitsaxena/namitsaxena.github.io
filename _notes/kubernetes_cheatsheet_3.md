---
layout: page
categories:
  - tech
title: kubectl cheat sheet - managing resources (declarative)
tags:
  - kubernetes
  - tech
  - cheatsheet
published: true
---

Resources that cannot be handled by kubectl cli(imperative commands) but need spec updates in yaml(declarative)
* ### Jobs
    * job.spec
        * activeDeadlineSeconds: terminate job after this many seconds (includes total number of retries)
            * contrast with job.spec.template.spec.activeDeadlineSeconds (container runs)
        * completions: desired number of successfully finished pods
        * backoffLimit: max tries before marking as failed (default: 6)
        * parallelism: 5
* ### Pod
    * securityContext at 2 levels
        * pod.spec.securityContext ([Doc](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-the-security-context-for-a-pod)): The security settings that you specify for a Pod apply to all Containers in the Pod.
            * fsGroup, runAsUser, runAsGroup
        * pod.spec.containers.securityContext ([Doc](https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/pods/security/security-context-2.yaml)): apply only to the individual Container, and they override settings made at the Pod level when there is overlap
            * runAsUser, runAsGroup, capabilities
    * pod.spec.serviceAccountName
    * pod.spec.containers.resources (also supported by imperative command - kubectl run)
        * pod.spec.containers.resources.requests: the scheduler uses this information to decide which node to place the Pod on
            * memory, cpu
        * pod.spec.containers.resources.limits: pod can consume more than request if node has available resources but it can not use more than the limit
            * memory, cpu
        * limitRange.spec.limits : default memory requests and limits for a namespace ([Doc](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/#create-a-limitrange-and-a-pod))
    * ports:
        * pod.spec.containers.ports.containerPort
            * containerPort as part of the pod definition is only for informational purposes.
                * Not specifying a port here DOES NOT prevent that port from being exposed. Any port which is listening on the default "0.0.0.0" address inside a container will be accessible from the network. 
                * Also, as noticed, if a pod/deployment is created and then ‘expose’ is used to create a service, it gets created and is accessible. However no container port is configured (this can be see by creating and pods/deployment and using expose to create a service)
            * [Should I configure the ports in the Kubernetes deployment?](https://medium.com/faun/should-i-configure-the-ports-in-kubernetes-deployment-c6b3817e495) - Medium
* ### Pod Volumes
    * pod.spec.volumes
        * name (common to all types)
        * pod.spec.volumes.hostPath ([Doc](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)) - path
        * emptyDir: {} ([Doc](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir))
        * ConfigMaps
            * pod.spec.volumes.configMap
                * name  (config map name) ([Doc](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#populate-a-volume-with-data-stored-in-a-configmap)) - configure all keys 
            * pod.spec.volumes.configMap.items ([Doc](https://kubernetes.io/docs/concepts/storage/volumes/#configmap)) - configure specific keys 
                * key, path
        * Secrets
            * pod.spec.volumes.secret
                * secretName ([Doc](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets)) - all keys Each key in the secret data map becomes the filename under mountPath.
            * pod.spec.volumes.secret.items ([Doc](https://kubernetes.io/docs/concepts/configuration/secret/#projection-of-secret-keys-to-specific-paths)) - configure specific keys
                * key, path
        * Persistent Volume Claims
            * pod.spec.volumes.persistentVolumeClaim
                * claimName
    * Mounting
        * pod.spec.containers.volumeMounts
            * pod.spec.containers[*].volumeMounts (container specific)
                * name, mountPath, readOnly: true
    * Persistent Volumes & Claims
        * Kubernetes binds PVC with best available PV based on storage class, capacity, volume and access mode, etc
        * Pod Volume uses PVC (which in turn is bound to PV) - see above
    * Misc
        * hostPath: Kubernetes supports hostPath for development and testing on a single-node cluster. A hostPath PersistentVolume uses a file or directory on the Node to emulate network-attached storage. ([Doc](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume))
            * for two pods to use the same hostPath volume (say even in PV), they must be scheduled on the same nodes
* ### Pod Environment Variables
    * Create SPECIFIC Env variables: 
        * pod.spec.containers.env -> name(name of env variable) and value (directly specify)
            * pod.spec.containers.env.valueFrom
                * pod.spec.containers.env.valueFrom.configMapKeyRef ([Doc](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-container-environment-variables-with-data-from-multiple-configmaps))
                    * name(name of CM), key(key name in CM), name of env variable is define above in env
                * pod.spec.containers.env.valueFrom.secretKeyRef ([Doc](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables))
                    * name(name of Secret), key(key name in Secret)
    * Create ALL env variables from some source
        * pod.spec.containers.envFrom
            * pod.spec.containers.envFrom.configMapRef ([Doc](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#configure-all-key-value-pairs-in-a-configmap-as-container-environment-variables))
                * name (config map name)
            * pod.spec.containers.envFrom.secretRef ([Doc](https://kubernetes.io/docs/concepts/configuration/secret/#use-case-as-container-environment-variables))
                * name (secret name)
* ### Pod - Probes
    * Types:
        * pod.spec.containers.readinessProbe: readiness probes to know when a container is ready to start accepting traffic. 
        * pod.spec.containers.livenessProbe: to know when to restart a container
        * pod.spec.containers.startupProbe: indicates that the Pod has successfully initialized. If specified, no other probes are executed until this completes successfully
    * Mechanisms: http, TCP, exec(command)
    * Failure and success thresholds can be defined
* ### Pod Scheduling
    * pod.spec.nodeName: is a request to schedule this pod onto a specific node
    * Node Selector - pod.spec.nodeSelector. Ex (the label Large must have been created - these are labels assigned to nodes): 
        * nodeSelector
            * Size: Large
    * Node Affinity: property of Pods that attracts them to a set of nodes it (hard or soft requirement), based on labels on the node
        * pod.spec.affinity.nodeAffinity
    * Taints & Tolerations
        * Taints: applied to a node allow a node to repel a set of pods
            * kubectl taint nodes node-name key=value:taint-effect
        * Tolerations: applied to pods, and allow (but do not require) the pods to schedule onto nodes with matching taints.
            * pod.spec.tolerations
            * A toleration "matches" a taint if the keys are the same and the effects are the same (and operator is Exists or Equals)
            * You can put multiple taints on the same node and multiple tolerations on the same pod. start with all of a node's taints, then ignore the ones for which the pod has a matching toleration; the remaining un-ignored taints have the indicated effects
* ReplicaSets
    * Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. Therefore, we recommend using Deployments instead of directly using ReplicaSets
    * a ReplicaSet is not limited to owning Pods specified by its template-- it can acquire other Pods (based on selector label)
        * You can remove Pods from a ReplicaSet by changing their labels
* ### Deployments
    * deployment.spec.strategy: RollingUpdate(default), Recreate
        * deployment.spec.strategy.rollingUpdate
            * maxSurge: maximum number of pods that can be scheduled above the desired number of pods (number/%)
            * maxUnavailable: maximum number of pods that can be unavailable during the update(number/%)
* ### Scaling
    * kubectl autoscale (can be used with Deployment, ReplicaSet, StatefulSet, or ReplicationController) - details in objects cheatsheet
        * creates [HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
* ### Services
    * svc.spec.type ([Doc](https://kubernetes.io/docs/concepts/services-networking/service/#defining-a-service)) 
* ### Ingress
* ### Network Policies
