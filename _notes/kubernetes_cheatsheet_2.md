---
layout: page
categories: [tech]
title: kubectl cheat sheet - managing objects
tags: [kubernetes, tech, cheatsheet]
---
* ### Resources
    * [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) | Kubernetes[kubernetes.io]  
* ### LISTING/Finding objects
    * kubectl get pods,svc
    * List from all namespaces: kg all --all-namespaces (or use -A)
    * kg pod,svc -o wide (wide format, more columns)
    * kg svc --show-labels
        * kubectl get po nginx --v=7 (or 8 or 9 - list pods with different levels of verbosity)
    * Filtering by labels
        * kg pods -l run=nginx
        * kg svc -l component=apiserver,provider=kubernetes
        * kubectl get po -l '!env’ (show pods that don’t have the 'env’ label)
            * creation_method!=manual 
            * env in (prod,devel)
            * env notin (prod,devel)
            * app=pc,rel=beta
    * To print without headers:  kg pods --no-headers
    * Printing custom columns, field
        * add two custom columns using jsonpath
            * kubectl get pod  -o custom-columns=CONTAINER:.spec.containers[0].name,IMAGE:.spec.containers[0].image
                * kg pod busybox -o jsonpath='{.spec.containers[0].name}'
        * kg pods -L labelname1,labelname2 (show specific labels in separate columns)
            * Ex: kg pods -L app (here app is an actual label name applied to the pods or some other object)
* ### Labelling & Annotations
    * Labelling
        * kubectl label pods bar color=red
            * any object can be labelled pods, namespaces, etc
        * label pod nginx1 nginx2 nginx3 app=v1 (labelling multiple pods)
        * kubectl label pod nginx2 app=v2 --overwrite (overwrite a label)
        * kubectl label pod nginx1 nginx2 nginx3 app- (Removing a label app from 3 nginx pods)
    * Annotations
        * kubectl annotate pod nginx1 description='my description'
        * kubectl annotate pod nginx1 description- (remove above annotation)
    * Notes:
        * Labels are strings (as such true/false won’t work without quoting (ex: ‘true’ )
* ### Deleting
    * kubectl delete -f obj.yaml (for an object created using apply)
    * Delete immediately: 
        * kubectl delete pod test1 --force (—grace-period not needed since 1.19)
            * kubectl delete pod nginx-test --force --grace-period=0 (—now doesn’t work with —force. It’s same as--grace-period=1)
    * kubectl delete all —all (delete all resources in current namespace)
    * kubectl delete all -l app=foo (delete all objects matching a label; useful when deleting objects created using kubetcl create, run, expose)
* ### Updating
    * kubectl edit (the yaml won’t be saved though)
    * kubectl delete, and then apply 
    * Updating image - we can directly update the image for certain objects like pods, replicasets, deployments, etc
        * kubectl set image pod/nginx-test nginx-test=nginx:1.17.1 (where nginx-test is also the name of the container)
    * Replacing
        * kubectl replace --force -f ./pod.json (force replace a pod delete and then re-create the resource - service outage)
    * Patch: useful for modifying a single property or a limited number of properties of a resource without having to edit its definition in a text editor
        * kubectl patch deployment kubia -p '{"spec": {"minReadySeconds": 10}}'
    * Scaling: Set a new size for a Deployment, ReplicaSet, Replication Controller, or StatefulSet
        * kubectl scale deployment httpd-frontend --replicas=5
        * kubectl scale —repliacas=6 rs.yml
        * kubectl autoscale deployment nginx-dep --min=5 --max=10 --cpu-percent=80
            * creates horizontalpodautoscaler (kg horizontalpodautoscaler OR kg hpa)
    * Rollouts: can be done on: deployments, daemonsets, statefulsets
        * See deployments below for more info
* ### Creating Resources/Objects
    * We can rely on declarative (kubectl apply) for everything but certain objects are also supported with the imperative approach
    * Imperative
        * kubectl run
            * creates pods
            * will add label run=<name of the resource created>
        * kubectl create
            * clusterrole, clusterrolebinding, configmap, cronjob, deployment, job, namespace, poddisruptionbudget, priorityclass, quota, role, rolebinding, secret, service, serviceaccount
            * will add label app=<name of the resource created
            * will add job-name=<jobname>
* ### Accessing/Executing
    * Access/Get shell  Shell on pod container:
        * kubectl exec -it nginx -- /bin/sh (inside first container)
        * kubectl exec mypod -c ruby-container -- date (inside mypod's ruby-container)
    * Access a temporary pod for debugging (also see pods section below)
        * kubectl run busybox --image=busybox -it --rm --restart=Never -- sh
    * kubectl exec deploy/mydeployment -- date (inside first pod’s first container)
    * kubectl exec svc/myservice -- date (inside the service’s first pod’s first container)
    * Copying files from a pod
        * kubectl cp <pod-name>:/path/to/remote/file /path/to/local/file
            * kubectl cp nginx-cm:/etc/passwd ./nginx-cm-passwd.txt
    * port-forward: Forward a local port to a port on the Pod
        * kubectl port-forward nginx-svc 8080:80 (we can access the pod’s port 80 on http://localhost:8080/)
            * the same can be done with deployment, replicaset, etc
    * kubectl attach my pod -i (TODO: didn’t work: If you don't see a command prompt, try pressing enter)
        * The kubectl attach command is similar to kubectl exec, but it attaches to the main process running in the container instead of running an additional one. It can attach to the main process run by the container, which is not always bash. As opposed to exec, which allows you to execute any process within the container (often: bash)
        * The container must be configured with tty: true and stdin: true. By default both of those values are false
* ### Pods
    * kubectl run nginx --image=nginx --restart=Never
        * --port=80 (only exposes port 80 on the container, no service is created)
        * --serviceaccount=myuser
        * -l app=v1 (or —labels)
        * --env=MYNAME=NSX
    * kubectl run busybox --image=busybox --restart=Never -- /bin/sh -c "sleep 3600" (run with sleep command)
        * This will end after an hour and it’s status would be "completed"
    * same as above but will delete the pod automatically when it exits
        * kubectl run busybox --image=busybox -it --rm --restart=Never -- /bin/sh -c 'echo hello world'
        * kubectl run busybox --image=busybox -it --rm --restart=Never -- sh (same but gets a shell into a temp pod)
    * container requests and limits
        * kubectl run nginx1 --image=nginx --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi' --dry-run=client -o yaml
* ### Jobs
    * kubectl create job hello-job --image=busybox --dry-run -o yaml -- echo "Hello I am from job" > hello-job.yaml
        * kubectl create job hi-job --image=busybox -- /bin/sh -c 'echo hello;sleep 30;echo world'
            * without sh and quotes, the ; will split other two into separate local commands
            * jobs create a pod with name as hi-job-v6wpv 
    * **CronJobs**
        * kubectl create cronjob myjob --image=busybox --schedule="*/1 * * * *" -- sh -c 'date; echo Hello'
        * Cron Syntax: * * * * * <command to execute> (min hr day(date) month weekday). See Unix Cheatsheet for details
* ### Service
    * Service name format: ServiceName.Namespace.Service.domain
    * expose: a resource as a new Kubernetes service.: Looks up a deployment, service, replica set, replication controller or pod by name and uses the selector for that resource as the selector for a new service on the specified port.
        * kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
    * kubectl run httpd —image=httpd:alpine —port 80 —expose (create pod and expose)
    * kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
    * kubectl expose deployment simple-webapp-deployment —name=webapp-service --port=8080 --target-port=8080 —type=NodePort —dry-run=client -o yaml
        * nodePort can’t be specified here, but you can do 'k edit service' and update the node port (or use create service - see below)
    * kubectl expose pod nginx --port=80 --name nginx-service --type=NodePort --dry-run=client -o yaml
    * kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
        * the selector is chosen/assumed based on the name of the service (i.e. app: cara if the service name is cara)
    * Testing: 
        * curl node01:30080 (assuming nodes are accessible - we are inside cluster - controplane or some node, and the service is nodePort)
* ### Deployments
    * kubectl create deployment httpd-frontend --image=httpd:2.4-alpine
    * kubectl scale deployment httpd-frontend --replicas=5
    * kubectl create -f kubia-deployment-v1.yaml —record 
        * —record: write the command executed in the resource annotation kubernetes.io/change-cause. The recorded change is useful for future introspection.
    * Rollouts
        * kubectl rollout status deployment/nginx-hello-deployment
            * kubectl rollout status deployment nginx --revision=1
        * kubectl rollout history deployment/nginx-hello-deployment
        * kubectl rollout undo deployment/my-deployment
            * kubectl rollout undo deployment nginx --to-revision=2 (rollback to a specific revision)
        * kubectl rollout pause deployment nginx-dep (if paused any new changes will not be rolled out until resumes)
        *  kubectl rollout resume  deploy nginx-dep 
* ### Replication Controllers
    * kubectl delete rc kubia --cascade=false (delete a rc but keep pods running)
* ### ConfigMaps & Secrets
    * Key loading options are similar for both
    * ConfigMaps
        * kubectl create configmap my-config --from-literal=key1=config1 --from-literal=key2=config2
        * kubectl create configmap my-config --from-file=config-file.conf (key name os filename, value is entire file content)
            * kubectl create configmap my-config --from-file=/path/to/dir
    * Secrets
        * kubectl create secret generic my-secret --from-literal=key1=supersecret 
            *  kg secrets my-secret -o jsonpath='{.data.key1}' | base64 —decode (read it back)
        * kubectl create secret generic mysecret2 --from-file=username 
            * from file: here file username contains the secret username (key: username, value:actual value)
            * kg secret mysecret2 -o jsonpath='{.data.username}' | base64 -D
    * Options
        * --from-file=config-file.conf (key name os filename, value is entire file content)
        * --from-env-file=keyval.txt (docker env file: each entry in file is separate key, file should have key=value per line)
        * kubectl delete cm dir2 dir3 dir4
* ### Service Accounts
    * kubectl create serviceaccount “dashboard-sa"
* ### Taints and Tolerations
    * kubectl taint nodes node-name key=value:taint-effect
        * To remove a taint: kubectl taint nodes node1 key1=value1:NoSchedule-
* Persistent Volumes and Claims
* Resource Quotas: sets aggregate quota restrictions enforced per namespace
    * kubectl create quota myrq --hard=cpu=1,memory=1G,pods=2

### Misc Advanced Topics
* Admission Web hooks
    * Notes
        * web hooks work even on pods in a different namespace (tested with grumpy example)
        * The admission web hook rely on admission controller plugins which need to be enabled for this to work
            * kube-apiserver -h | grep enable-admission-plugins
                * the above command runs in tube-apiserver container. See Kubernetes Setup and Guide for how to do this
                * this should list MutatingAdmissionWebhook and ValidatingAdmissionWebhook
                * See the link for how to enabled/disable this if needed
    * Commands
    * kubectl get ValidatingWebhookConfiguration
    * kubectl get MutatingWebhookConfiguration
    * kubectl delete ValidatingWebhookConfiguration grumpy
### * Misc
    * Container restartPolicy (within a pod):
        * Always: means that the container will be restarted even if it exited with a zero exit code (i.e. successfully). This is useful when you don't care why the container exited, you just want to make sure that it is always running (e.g. a web server). This is the default.
        * OnFailure means that the container will only be restarted if it exited with a non-zero exit code
        * Never means that the container will not be restarted regardless of why it exited.