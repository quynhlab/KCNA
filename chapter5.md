#KCNA 5: kubernetes in practice

### 1. Introduction<a href="#1__7" id="1__7"></a>

In this chapter, we will learn about the different Kubernetes objects, their purpose and how to interact with them. \
After setting up the cluster or using an existing cluster, we can start deploying some workloads. The smallest unit of computation in Kubernetes is not a container, but a Pod object. That said, Pods are not the only abstraction we use for workloads. Kubernetes has various workload objects to control how pods are deployed, scaled and managed. \
Deploying workloads is not the only task a developer or administrator must perform. Kubernetes provides solutions to some of the inherent problems of containers and orchestration, such as configuration management, cross-node networking, external traffic routing, load balancing, or scaling of pods.

### 2. Learning Objectives <a href="#2__13" id="2__13"></a>

By the end of this chapter, you should be able to:

* Explain what a Kubernetes object is and how to describe it.
* Discuss the concept of pods and the problems they solve.
* Learn how to use workload resources to scale and schedule pods.
* Learn how to abstract Pods with services, and how to expose them.

One of the core concepts of Kubernetes is to provide a number of abstract resources (also called objects) that you can use to describe how a workload should be handled. Some of them deal with container orchestration issues, such as scheduling and self-healing, and others address some of the problems inherent in containers. \
Kubernetes objects can be divided into workload-oriented objects (for handling container workloads) and infrastructure-oriented objects (for example, for handling configuration, networking, and security). Some of these objects can be placed in a namespace, while others can be used across the entire cluster. \
As users, we can describe these objects in YAML, a popular data serialization language, and send them to the api server, where they are validated before being created.

````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
````

Fields highlighted in red are required fields. They include:

* `apiVersion`: Every object can be versioned. This means that the data structure of an object can change between versions.
* `kind`: The type of object that should be created. \
  `metadata`: data that can be used to identify it. Every object requires a name and must be unique. If you need multiple objects with the same name, you can use namespaces.
* `spec`: The description of the object. Here you can describe the state you want. Be careful, as the structure of the object may change with its version!

Creating, modifying or deleting an object is just an intent record, where you describe the state your object should be in, you don't actively start pods or even containers like you would do on your local machine, and get direct feedback if it worked or not.

### 4. Interacting with Kubernetes <a href="#4_kubernetes_59" id="4_kubernetes_59"></a>

To access the API, users can use the official command line interface client called kubectl. Let's take a look at some basic commands that Kubernetes uses on a daily basis. \
Note: You can learn how to install kubectl in the official documentation. \
You can list the objects available in the cluster with the following command:

````
$ kubectl api-resources

NAME SHORTNAMES APIVERSION NAMESPACED KIND
...
configmaps cm v1 true ConfigMap
...
namespaces ns v1 false Namespace
nodes no v1 false Node
persistentvolumeclaims pvc v1 true PersistentVolumeClaim
...
pods po v1 true Pod
...
services svc v1 true Service
````

Notice how objects use short names. This is useful for objects with long names such as `configmaps` or `persistentvolumeclaims`. The table also shows which objects have namespaces and their available versions. \
If you want to learn more about objects, kubectl has a built-in `explain` function!\
Let's learn more about pods:

````
$ kubectl explain pod

KIND: Pod
VERSION: v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is     
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion <string>     
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal         
     value, and may reject unrecognized values.
...
   kind <string>
...
   metadata <Object>
...
   spec <Object>
````

To learn more about pod specifications, you can dig into object definitions. Use the following format: `<type>.<fieldName>[.<fieldName>]`.

````
$ kubectl explain pod.spec

KIND: Pod
VERSION: v1

RESOURCE: spec <Object>  

DESCRIPTION:
     Specification of the desired behavior of the pod. More info:

https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status      

     PodSpec is a description of a pod.

FIELDS:
   activeDeadlineSeconds <integer>     
     Optional duration in seconds the pod may be active on the node relative to       
     StartTime before the system will actively try to mark it failed and kill         
     associated containers. Value must be a positive integer.

   affinity <object>     
     If specified, the pod's scheduling constraints

   automountServiceAccountToken <boolean>     
     AutomountServiceAccountToken indicates whether a service account token           
     should be automatically mounted.

   containers <[]Object> -required-
...
````

Let's look at the basic kubectl commands. You can use the --help flag to view them:

````
$ kubectl --help

kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/overview/

Basic Commands (Beginner):
  create Create a resource from a file or from stdin
  expose Take a replication controller, service, deployment or pod and expose it as a new Kubernetes service
  run Run a particular image on the cluster
  set Set specific features on objects

Basic Commands (Intermediate):
  explain Get documentation for a resource
  get Display one or many resources
  edit Edit a resource on the server
  delete Delete resources by file names, stdin, resources and names, or by resources and label selector
````

To create an object from a YAML file in Kubernetes, you can use the following command:

````
kubectl create -f <your-file>.yaml
````

Kubernetes has many GUIs and dashboards that allow visual interaction with the cluster.

![Insert image description here](https://img-blog.csdnimg.cn/80b0aef0e6b54f2a87dabfbf219b8e24.png)\
Screenshot of the official Kubernetes dashboard\
Other tools to interact with Kubernetes:

* [kubernetes/dashboard](https://github.com/kubernetes/dashboard)
* [derailed/k9s](https://github.com/derailed/k9s)
* [Lens](https://k8slens.dev)
* [VMware Tanzu Octant](https://github.com/vmware-tanzu/octant)

While there are many CLI tools and guis, there are also advanced tools that allow creating templates and packaging Kubernetes objects. Perhaps the most used tool for Kubernetes today is Helm. \
[Helm](https://helm.sh) is a package manager for Kubernetes that allows simpler updates and interaction with objects. Helm encapsulates Kubernetes objects in so-called Charts, which can be shared with others via a registry. To get started with Kubernetes, you can search [ArtifactHub](https://artifacthub.io) to find your favorite package, ready to deploy.

#### 4.1 Demo: kubectl <a href="#41_demo_kubectl_180" id="41_demo_kubectl_180"></a>

* [kubectl command](https://blog.csdn.net/xixihahalelehehe/article/details/107714611?ops\_request\_misc=%257B%2522request%255Fid%2522%253A%2522164734401216780261970494%2522%252C%2522scm%252C%2522 %253A%252220140713.130102334.pc%255Fblog.%2522%257D\&request\_id=164734401216780261970494\&biz\_id=0\&spm=1018.2226.3001.4450)

### 5. Pod Concept <a href="#5_pod__185" id="5_pod__185"></a>

The most important objects in Kubernetes are Pods. A pod describes a unit of one or more containers that share a namespace and isolation layer of cgroups. It is the smallest deployable unit in Kubernetes, which also means that Kubernetes does not interact with containers directly. The pod concept was introduced to allow a combination of running multiple processes that depend on each other. All containers in a pod share an IP address and can be shared through the file system.

![Insert image description here](https://img-blog.csdnimg.cn/2066df4484e14e76a9c650f8d328e576.png)\
Multiple containers share a namespace to form a pod\
Here is an example of a simple Pod object with two containers:

````
apiVersion: v1
kind: Pod
metadata:
  name: nginx-with-sidecar
spec:
  containers:
  - name: nginx
    image: nginx:1.19
    ports:
    - containerPort: 80
  - name: count
    image: busybox:1.34
    args: [/bin/sh, -c,
            'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done']
````

You can add any number of containers to the main application. Be careful though, as you lose the ability to scale them individually! Using a second container that backs the main application is called a **sidecar container**. \
All defined containers are started at the same time, in no order, but you can also use [initContainers](https://kubernetes.io/en/docs/concepts/workloads/pods/init-containers/) in the main app Start the container before the program starts. In this example, the init container `init-myservice` is trying to reach another service. Once done, the main container will start.

````
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
````

Be sure to browse the documentation on pods as there are many more settings to discover. Examples of some important settings that can be set for each container in a Pod are as follows:

* [resources](https://ghostwritten.blog.csdn.net/article/details/112524133): Set a resource request and the maximum limit of CPU and memory
* [livenessProbe](https://blog.csdn.net/xixihahalelehehe/article/details/108561740?ops\_request\_misc=%257B%2522request%255Fid%2522%253A%2522164734455316780366546099%2522%252C%2522%cm% 253A%252220140713.130102334.pc%255Fblog.%2522%257D\&request\_id=164734455316780366546099\&biz\_id=0\&spm=1018.2226.3001.4450): Configure a health check that periodically checks whether the application is still active. If the check fails, the container can be restarted.
* [securityContext](https://blog.csdn.net/xixihahalelehehe/article/details/108539153?ops\_request\_misc=%257B%2522request%255Fid%2522%253A%2522164734458716780271923305%2522%252C%2522scm% 253A%252220140713.130102334.pc%255Fblog.%2522%257D\&request\_id=164734458716780271923305\&biz\_id=0\&spm=1018.2226.3001.4450): Set user and group settings, and kernel features.

#### 5.1 Demo: Pods <a href="#51_demo_pods_235" id="51_demo_pods_235"></a>

````
docker run -d nginx:1.19
kubectl run nginx --image=nginx:1.19
kubectl get pods
kubectl describe pod nginx
#get IP
curl http://IP
````

### 6. Load Balancer <a href="#6__246" id="6__246"></a>

In a container orchestration platform, just using Pods is not flexible enough. For example, if a Pod is lost due to a node failure, it is gone forever. To ensure that a defined number of Pod replicas are always running, we can use a controller object to manage the Pods for us.

* `ReplicaSet`\
  A controller object that ensures the desired number of pods are running at any given time. A replicaset can be used to scale out an application and increase its availability. They do this by starting multiple copies of a pod definition.
* `Deployment`\
  The most feature-rich object in Kubernetes. Deployments can be used to describe the complete application lifecycle, and they do this by managing multiple `replicasets` that are updated when the application is changed, for example, to provide a new container image. Deployments are great for running stateless applications in Kubernetes.
* `StatefulSet`\
  StatefulSets can be used to run stateful applications like databases on Kubernetes, which was considered a bad practice for a long time. Stateful applications have special needs that don't fit into the ephemeral nature of pods and containers. Unlike deployments, StatefulSets try to preserve the IP addresses of pods and give them a stable name, persistent storage, and more graceful scaling and update handling.
* `DaemonSet`\
  Make sure a replica of the Pod is running on all (or some) nodes in the cluster. The daemon set is great for running infrastructure-related workloads, such as monitoring or logging tools.
* `Job`\
  Create one or more Pods that execute a task, then terminate the task. Job objects are great for running one-off scripts like database migrations or administrative tasks.
* `CronJob`\
  CronJobs add time-based configuration for jobs. This allows to run Jobs on a regular basis, such as a backup job every night at 4pm.

Interactive Tutorial - Deploying an Application and Exploring It\
In part 2 of the interactive tutorial provided by the Kubernetes documentation, you can learn how to [deploy an application] in a Minikube cluster (https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy- intro/). \
Apply what you've learned from "Interacting with Kubernetes" in the third part of the interactive tutorial [Explore Your Application](https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-intro /).

#### 6.2 demo: pods, replicas, deployments <a href="#62_demo_podreplicatsdeployments_268" id="62_demo_podreplicatsdeployments_268"></a>

````
apiVersion: v1
kind: Pod
metadata:
  name: simple-nginx-pod
  labels:
    role: myrole
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
````

````
kubectl apply -f simple-nginx-pod.yaml
````

deployment of replicas

````
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      apps: nginx
  template:
    metadata:
      labels:
        apps: nginx
    spec:
      containers:
      - name: web
        image: nginx
        ports:
        - containerPort: 80
````

````
$ kubectl apply -f replicas.yaml
$ k get pods
NAME READY STATUS RESTARTS AGE
nginx-5psrm 1/1 Running 0 2m12s
nginx-68x8p 1/1 Running 0 2m12s
nginx-q9zlq 1/1 Running 0 2m12s


$ kubectl scale --replicas=4 rs/nginx
````

deployment deployment

````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
````

````
$ k apply -f deployment.yaml
$ k get pods
NAME READY STATUS RESTARTS AGE
nginx-deployment-66b6c48dd5-55jrb 1/1 Running 0 2m35s
nginx-deployment-66b6c48dd5-6x9cj 1/1 Running 0 2m35s
nginx-deployment-66b6c48dd5-pj7qr 1/1 Running 0 2m35s


$ k scale --replicas=4 deployment/nginx-deployment

$ k get pods
NAME READY STATUS RESTARTS AGE
nginx-deployment-66b6c48dd5-55jrb 1/1 Running 0 4m15s
nginx-deployment-66b6c48dd5-6x9cj 1/1 Running 0 4m15s
nginx-deployment-66b6c48dd5-cxv8g 1/1 Running 0 3s
nginx-deployment-66b6c48dd5-pj7qr 1/1 Running 0 4m15s

$ k set image deployment/nginx-deployment nginx=nginx:1.20
````

### 7. Network <a href="#7__372" id="7__372"></a>

Since a large number of Pods require a lot of manual network configuration, we can use the `Service` and `Ingress` objects to define and abstract the network

* `ClusterIP`\
  The most common type of service. `ClusterIP` is a virtual IP inside `Kubernetes` that can be used as a single endpoint for a group of pods. This service type can be used as a round robin load balancer.

![Insert picture description here](https://img-blog.csdnimg.cn/de1b650ea01e4cb2b126356c8c4cdc38.png#pic\_center)

* `NodePort`\
  The NodePort service type extends `ClusterIP` by adding simple routing rules. It opens a port (between 30000-32767 by default) on each node in the cluster and maps it to ClusterIP. This service type allows external traffic to be routed to the cluster.
* `loadbalance`\
  The LoadBalancer service type extends NodePort by deploying an external LoadBalancer instance. This will only work if you are in an environment that has an API to configure LoadBalancer instances, such as GCP, AWS, Azure or even OpenStack.
* `ExternalName`\
  A special type of service without any routing. `ExternalName` creates a DNS alias using the Kubernetes internal DNS server. You can use it to create a simple alias to resolve a rather complex hostname like: `my-cool-database-az1-uid123.cloud-provider-i-like.com`. This is especially useful if you want to fetch external resources from a `Kubernetes` cluster.

![Insert picture description here](https://img-blog.csdnimg.cn/ebfc07d9f3ae488eb6e7b1f561578b43.png#pic\_center)

ClusterIP, NodePort and LoadBalancer extend each other

If you need more flexibility to expose your application, you can use an `Ingress` object. Ingress provides a way to expose HTTP and HTTPS routes for services within the cluster from outside the cluster. It does this by configuring routing rules, which users can set and implement through ingress controllers.

![Insert picture description here](https://img-blog.csdnimg.cn/b4033da09fec4767ba8d62e530e67cda.png#pic\_center)

An example of an Ingress sending all traffic to a Service, taken from [Kubernetes documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/)

Standard features of an ingress controller may include:

* LoadBalancing
* TLS offloading/termination
* Name-based virtual hosting
* Path-based routing

Many ingress controllers offer even more functionality, such as:

* Redirects
*Custom errors
* Authentication
* Session affinity
* Monitoring
* Logging
* Weighted routing
* Rate limiting.

Kubernetes also provides an intra-cluster firewall with the concept of NetworkPolicy. `NetworkPolicies` is a simple IP firewall (OSI layer 3 or 4) that can control traffic based on rules. You can define rules for incoming (incoming) and outgoing (egress) traffic. A typical use case for NetworkPolicies is to restrict traffic between two different namespaces.

Interactive Tutorial - Showcase Your Application\
You can now learn how to expose an application using a Service in part 4 of the interactive tutorial provided by the Kubernetes documentation (https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-intro/) .

#### 7.1 demo <a href="#71_demo_422" id="71_demo_422"></a>

````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
````

````
$ k apply -f nginx-deployment.yaml
$ k expose deployment nginx-deployment 80
$ k get svc
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
kubernetes ClusterIP 10.96.0.1 <none> 443/TCP 51d
nginx-deployment ClusterIP 10.101.106.248 <none> 80/TCP 8s
$ curl 10.101.106.248:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
````

### 8. Volume & Storage <a href="#8_volume__storage_482" id="8_volume__storage_482"></a>

As mentioned earlier, containers are not designed with persistent storage in mind, especially when storage spans multiple nodes. Kubernetes introduces some solutions, but please note that these solutions do not automatically remove all the complexities of using containers to manage storage. \
Containers already have the concept of mounting volumes, but since we're not using containers directly, Kubernetes treats volumes as part of a Pod, just like containers. \
The following is an example of a `hostPath` volume mount, similar to the host mount introduced by Docker:

````
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    hostPath:
      # directory location on host
      path: /data
      # this field is optional
      type: Directory
````

![Insert picture description here](https://img-blog.csdnimg.cn/63c0ca09884d4d37854f1c14046900ab.png#pic\_center)\
Volumes allow data to be shared among multiple containers in the same Pod. This concept allows great flexibility when you want to use sidecar mode. Their second use is to prevent data loss if a Pod crashes and is restarted on the same node. The pod starts in a clean state, but all data is lost unless written to the volume. \
Unfortunately, clustered environments with multiple servers require more flexibility in terms of persistent storage. Depending on the environment, we can use tools like [Amazon EBS](https://aws.amazon.com/ebs/), [Google Persistent Disks](https://cloud.google.com/persistent-disk), [ Azure Disk storage](https://azure.microsoft.com/en-us/services/storage/disks/), you can also use cloud block storage like [Ceph](https://ceph.io/en/ ), storage systems like [GlusterFS](https://www.gluster.org), or more traditional systems like NFS. \
These are just a few examples of storage that can be used in Kubernetes. To make the user experience more uniform, Kubernetes uses the Container Storage Interface (CSI), which allows storage vendors to write a plugin (storage driver) that can be used in Kubernetes.

To use this abstraction, we have two more objects that we can use:

* PersistentVolumes (PV)\
  An abstract description of a memory slice. The object configuration contains the type of volume, volume size, access mode and unique identifier, and information on how to mount it.
* PersistentVolumeClaims (PVC)\
  User request for storage. If the cluster has multiple persistent volumes, the user can create a PVC and reserve a persistent volume according to the user's needs.

````
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-pv
spec:
  capacity:
    storage: 50Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  csi:
    driver: ebs.csi.aws.com
    volumeHandle: vol-05786ec9ec9526b67
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
    - name: app
      image: centos
      command: ["/bin/sh"]
      args:
        ["-c", "while true; do echo $(date -u) >> /data/out.txt; sleep 5; done"]
      volumeMounts:
        - name: persistent-storage
          mountPath: /data
  volumes:
    - name: persistent-storage
      persistentVolumeClaim:
        claimName: ebs-claim
````

This example shows a `PersistentVolume` that uses an AWS EBS volume implemented using the CSI driver. After the PersistentVolume is configured, developers can use the PersistentVolumeClaim to reserve it. The final step is to use the PVC as the volume in the Pod, just like the hostPath example we saw earlier. \
Storage clusters can be operated directly in Kubernetes. Projects like [Rook](https://rook.io) provide cloud-native storage business orchestration and integration with battle-tested storage solutions like Ceph. \
![Insert picture description here](https://img-blog.csdnimg.cn/485246bfc6ff4e809733f5384afc2eb5.png#pic\_center)

Rook Architecture, [Retrieved from Rook Documentation](https://rook.io/docs/rook/v1.7/ceph-storage.html)

### 9. Configuration object <a href="#9__571" id="9__571"></a>

[The 12factor app recommends storing the configuration in the environment](https://12factor.net/config). But what exactly does that mean? It takes more than just application code and some libraries to run an application. Applications have configuration files to connect to other services, databases, storage systems or caches, which require configuration like connection strings. \
It is considered bad practice to incorporate configuration directly into container builds. Any configuration change requires rebuilding the entire image and redeploying the entire container or pod. This problem only gets worse when using multiple environments (dev, staging, production) and building images for each. The 12factor app explains this in more detail: [Dev/prod parity](https://12factor.net/dev-prod-parity). \
In Kubernetes, this problem is solved by decoupling configuration from Pods using ConfigMaps. ConfigMaps can be used to store entire configuration files or variables as key-value pairs. There are two possible ways to use a ConfigMap:

* Mount the ConfigMap as a volume in the Pod
* Map the variables in the ConfigMap to the environment variables in the Pod.

Here is an example of a ConfigMap containing nginx configuration:

````
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
data:
  nginx.conf: |
    user nginx;
    worker_processes 3;
    error_log /var/log/nginx/error.log;
...
      server {
          listen 80;
          server_name _;
          location / {
              root html;
              index index.html index.htm; } } }
````

Once the ConfigMap is created, you can use it in the Pod:

````
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.19
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: /etc/nginx
      name: nginx-conf
  volumes:
  - name: nginx-conf
    configMap:
      name: nginx-conf
````

From the beginning, Kubernetes also provided an object to store sensitive information such as passwords, keys or other credentials. These objects are called [Secrets](https://kubernetes.io/en/docs/concepts/configuration/secret/#using-secrets). Secrets are very related to ConfigMaps, basically their only difference is that secrets are base64 encoded. \
There's been a lot of debate about the risks of using "secret" because "secret" (contrary to the name) is not considered secure. In cloud-native environments, purpose-built secret management tools have emerged that integrate well with Kubernetes. [HashiCorp Vault](https://www.vaultproject.io) is an example.

### 10. Autoscaling <a href="#10_autoscaling_624" id="10_autoscaling_624"></a>

Automatic scaling mechanism

* [Horizontal Pod Autoscaler (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)\
  Horizontal Pod Autoscaler (HPA) is the most commonly used autoscaler in Kubernetes. HPA can monitor deployments or ReplicaSets and increase the number of replicas when a certain threshold is reached. Imaging Pods can use 500MiB of memory and you configure a threshold of 80%. If the utilization exceeds 400MiB (80%), the second Pod will be scheduled. Now your capacity is 1000MiB. If 800MiB is used, the third pod will be scheduled, and so on.
* [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler)\
  Of course, if the cluster capacity is fixed, it doesn't make sense to spin up more and more pod replicas. If demand increases, the Cluster Autoscaler can add new worker nodes to the cluster. The cluster autoscaler works in conjunction with the horizontal autoscaler.
* [Vertical Pod Autoscaler](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)\
  The Vertical Pod Autoscaler is relatively new and allows pods to dynamically increase resource requests and limits. As mentioned before, vertical scaling is limited by node capacity.

Unfortunately, Kubernetes' (horizontal) autoscaling is not available out of the box and requires the installation of an add-on called [metrics-server](https://github.com/kubernetes-sigs/metrics-server).

However, it is possible to replace the metrics server with the [Prometheus Adapter](https://github.com/kubernetes-sigs/prometheus-adapter) of the Kubernetes Metrics api. prometheus-adapter allows you to use custom metrics in Kubernetes and scale up or down based on factors like the number of requests or users on the system. \
Projects like [KEDA](https://keda.sh) can scale Kubernetes workloads based on events triggered by external systems, rather than relying solely on metrics. KEDA, short for kubernetes-based event-driven automated scaler, was launched in 2019 as a partnership between Microsoft and Red Hat. Similar to HPA, KEDA can scale deployments, replica sets, pods, etc., as well as other objects such as Kubernetes jobs. With a large selection of out-of-the-box scalers, KEDA can scale to special triggers like database queries, or even the number of pods in a Kubernetes cluster.

Interactive Tutorial - Scaling Your Application\
In the fifth part of the interactive tutorial: Running Multiple Instances of Your Application, you can learn [How to Scale Your Application Manually](https://kubernetes.io/docs/tutorials/kubernetes-basics/scale/scale-intro /).

### 11. Additional Resources <a href="#11_additional_resources_643" id="11_additional_resources_643"></a>

Learn more about…

Differences between Containers and Pods

* [What are Kubernetes Pods Anyway?](https://www.ianlewis.org/en/what-are-kubernetes-pods-anyway), by Ian Lewis (2017)
* [Containers vs. Pods](https://iximiuz.com/en/posts/containers-vs-pods/) - Taking a Deeper Look, by Ivan Velichko (2021)

kubectl tips & tricks

* [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

Storage and CSI in Kubernetes

* [Container Storage Interface (CSI) for Kubernetes GA](https://kubernetes.io/blog/2019/01/15/container-storage-interface-ga/), by Saad Ali\
  (2019)
* [Kubernetes Storage: Ephemeral Inline Volumes, Volume Cloning,Snapshots and more!](https://www.inovex.de/de/blog/kubernetes-storage-volume-cloning-ephemeral-inline-volumes-snapshots/), by Henning Eggers (2020)

Autoscaling in Kubernetes

* [Architecting Kubernetes clusters - choosing the best autoscaling strategy](https://learnk8s.io/kubernetes-autoscaling-strategies), by Daniele Polencic (2021)

***
