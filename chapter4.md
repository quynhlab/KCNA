#KCNA 4: kubernetes basics

@\[toc]

***

### 1. Introduction to Kubernetes

Kubernetes is a very popular open source container orchestration platform that can be used to automate the deployment, scaling and management of containerized workloads. In this chapter, we will discover the basic architecture of a Kubernetes cluster and its components. To learn more Kubernetes basics, you can take the Linux Foundation's [Free Introduction to Kubernetes (LFS158x) course](https://training.linuxfoundation.org/training/introduction-to-kubernetes/#). Originally designed and developed by Google, Kubernetes was open-sourced in 2014, with version 1.0 being donated to the newly formed cloud-native computing.

### 2. Learning Objectives

By the end of this chapter, you should be able to:

1. Discuss the basic architecture of Kubernetes.
2. Explain the different components of the control plane and worker nodes.
3. Learn how to get started with a Kubernetes setup.
4. Discuss how Kubernetes runs containers.
5. Discuss Kubernetes scheduling concepts.

### 3. Kubernetes Architecture

Kubernetes is often used as a cluster, which means it spans multiple servers working on different tasks and distributes the load of the system. This was an initial design decision based on Google's needs, with billions of containers launching every week. Given the high vertical scalability of Kubernetes, it is possible to have clusters of thousands of server nodes across multiple data centers and regions. From a high-level perspective, a Kubernetes cluster consists of two distinct server node types:

* Control plane nodes(s) These are the brains of the action. Control plane nodes contain various components that manage the cluster and control various tasks such as deployment, scheduling, and self-healing of containerized workloads.
* Worker Nodes Worker nodes are where applications run in the cluster. This is the only job for worker nodes, they do not have any further logic to implement. Their behavior, such as whether they should start a container, is completely controlled by the control plane nodes.

![Insert picture description here](https://img-blog.csdnimg.cn/3f32971c0ea746eb9b35d33ae3dc6365.png#pic\_center)

The Kubernetes architecture is similar to the microservices architecture you choose for your own applications, Kubernetes incorporates multiple smaller services that need to be installed on nodes.

master node components:

* `kube-apiserver`: This is the core view of Kubernetes. All other components interact with the api server, which is where users access the cluster.
* `etcd`: The database that holds the cluster state. etcd is an independent project and not an official part of Kubernetes.
* `kube-scheduler`: When a new workload should be scheduled, kube-scheduler will select an appropriate worker node based on different properties such as CPU and memory.
* `kube-controller-manager`: Contains various non-terminating control loops for managing the state of the cluster. For example, one of the control loops ensures that the desired number of your application is always available.

node node components:

* Container runtime: The container runtime is responsible for running containers on worker nodes. Docker was the most popular choice for a long time, but is now being superseded by other runtimes like [containerd](https://containerd.io).
* kubelet: A small proxy that runs on each worker node in the cluster. The kubelet talks to the api server and the container runtime to handle the final stage of starting the container.
* kube-proxy: A network proxy that handles communication inside and outside the cluster. If possible, the kube proxy tries to rely on the networking capabilities of the underlying operating system instead of managing the traffic itself.

Notably, this design allows applications already launched on worker nodes to continue running in the event that the control plane is unavailable. Although many important functions, such as scaling, scheduling new applications, etc., will not be possible to implement when the control plane is offline. Kubernetes also has a concept of namespaces, not to be confused with the kernel namespaces used to isolate containers. Kubernetes namespaces can be used to divide a cluster into multiple virtual clusters, which can be used for multi-tenancy when using mul.

### 4. Kubernetes build

Setting up a Kubernetes cluster can be accomplished in many different ways. With the right tools, creating a test "cluster" is very easy:

* [Minikube](https://minikube.sigs.k8s.io/docs/)
* [kind](https://kind.sigs.k8s.io)
* [MicroK8s](https://microk8s.io)

If you want to set up a production-grade cluster on your own hardware or virtual machines, you can choose one of the various installers:

* [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)
* [kops](https://github.com/kubernetes/kops)
* [kubespray](https://github.com/kubernetes-sigs/kubespray)

Some vendors are starting to package Kubernetes into a distribution and even offer commercial support:

* [Rancher](https://rancher.com)
* [k3s](https://k3s.io)
* [OpenShift](https://www.redhat.com/en/technologies/cloud-computing/openshift)
* [VMWare Tanzu](https://tanzu.vmware.com/tanzu)

These distributions often choose a stubborn approach and provide additional tools when using Kubernetes as a central part of their framework. If you don't want to install and manage it yourself, you can use it from your cloud provider:

* [Amazon (EKS)](https://aws.amazon.com/eks/)
* [Google (GKE)](https://cloud.google.com/kubernetes-engine)
* [Microsoft (AKS)](https://azure.microsoft.com/en-us/services/kubernetes-service/)
* [DigitalOcean (DOKS)](https://www.digitalocean.com/products/kubernetes)

Interactive Tutorial - Creating a Cluster In this interactive tutorial, you can learn how to use Minikube [build your own Kubernetes cluster](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster -intro/).

### 5. Kubernetes API

The `Kubernetes API` is the most important component in a Kubernetes cluster. Without it, it is impossible to communicate with the cluster, every user and every component of the cluster itself needs the api-server. An overview of access control, retrieved from [Kubernetes documentation](https://kubernetes.io/docs/concepts/security/controlling-access/) Before a request is processed by Kubernetes, it must go through three phases:

* The authentication requester needs to provide a way to authenticate against the API. Typically digitally signed certificates (X.509) or external identity management systems are used. Kubernetes users are always managed externally. Service accounts can be used to authenticate technical users.
* Authorization It determines what the requester is allowed to do. In Kubernetes, this can be achieved through role-based access control (RBAC).
* Allow control [add link description](https://cri-o.io) In the final step, the admission controller can be used to modify or validate the request. For example, if a user tries to use a container graph from an untrusted registry

An admission controller may block this request. Tools like [Open Policy Agent](https://www.openpolicyagent.org) can be used for external management [Admission Control](https://kubernetes.io/docs/concepts/security/controlling-access/) . Like many other APIs, the Kubernetes API is implemented as a RESTful interface exposed over HTTPS. Through this API, users or services can create, modify, delete or retrieve resources residing in Kubernetes.

### 6. Running containers on Kubernetes

How is running a container on a local machine different from running a container in Kubernetes? In Kubernetes, instead of launching a container directly, Pods are defined as the smallest unit of computation, which Kubernetes turns into a running container. We'll learn more about Pods later, but for now think of it as a wrapper around a container. When creating a Pod object in Kubernetes, there are several components involved before getting the container running the node. Here is an example using Docker:

To allow the use of other container runtimes instead of Docker, Kubernetes introduced the [Container Runtime Interface (CRI)](https://kubernetes.io/blog/2016/12/container-runtime-interface-cri-in- kubernetes/).

* `containerd`: [Containerd](https://containerd.io) is a lightweight, high-performance implementation of running containers. Arguably the most popular container runtime out there. It is used by all major cloud providers of Kubernetes-as-a-Service offerings.
*CRI-O: [CRI-O](https://cri-o.io) was created by Red Hat with a similar codebase closely related to podman and buildah.
*Docker: This standard has been around for a long time, but was never really used for container orchestration. The use of Docker as a Kubernetes runtime has been deprecated and will be removed in Kubernetes 1.23. [Kubernetes has a great blog post](https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/) that answers all your questions on the subject.

Creating containers with containerd is much simpler than with Docker The idea of ​​`containerd` and `CRI-O` is very simple: provide a runtime that contains only what is absolutely necessary to run the container. However, they have some additional features, such as the ability to integrate with container runtime sandbox tools. These tools try to solve the security problems that come with sharing the kernel among multiple containers. The most commonly used tools are:

* [gvisor](https://github.com/google/gvisor): Created by Google, provides an application kernel that sits between the containerized process and the host kernel.
* [Kata Containers](https://katacontainers.io): Provides a secure runtime for lightweight virtual machines, but behaves like containers.

### 7. Networking

Kubernetes networking can be very complex and difficult to understand. Many of these concepts are unrelated to kubernetes and are covered in the container orchestration chapter. Likewise, we have to deal with a large number of containers that need to communicate across a large number of nodes. Kubernetes distinguishes between four network problems that need to be solved:

1. Container-to-Container communication This can be solved by the Pod concept, which we will learn about later.
2. Pod-to-Pod communication This can be solved by overlay network.
3. Pod-to-Service communication is implemented on the node through kube-proxy and packet filtering External-to-Service communication is implemented on the node through kube-proxy and packet filter.

There are different ways of implementing networking in Kubernetes, but there are also three important requirements:

* All pods can communicate with each other across nodes.
* All nodes can communicate with all pods.
* No Network Address Translation (NAT).

To achieve networking, you can choose from a variety of network providers:

* [Project Calico](https://www.tigera.io/project-calico/)
* [Weave](https://www.weave.works/oss/net/)
* [Cilium](https://cilium.io)

In Kubernetes, each Pod has its own IP address, so no manual configuration is required. Additionally, most Kubernetes setups include a DNS server add-on called [core-dns](https://kubernetes.io/docs/tasks/administer-cluster/coredns/), which provides service discovery and Name resolution.

### 8. Scheduling

In its most basic form, scheduling is a subclass of container orchestration that describes the process of automatically selecting the correct (worker) nodes to run container workloads. In the past, scheduling was more of a manual task, with system administrators choosing the right server for an application by keeping track of available servers, their capacity, and other attributes such as their location.

In a Kubernetes cluster, the kube scheduler is the component that makes scheduling decisions, but is not responsible for actually launching the workload. The scheduling process in Kubernetes always starts when a new Pod object is created. Remember, Kubernetes uses a declarative approach where Pods are just described first and then the scheduler chooses a node where the Pods will actually be started by the kubelet and the container runtime.

A common misconception about Kubernetes is that it has some form of "artificial intelligence" to analyze workloads and move pods around based on resource consumption, workload type, and other factors. The fact is that the user has to provide information about the needs of the application, including requests for CPU and memory and the properties of the nodes. For example, a user can request that their application requires two CPU cores, 4g of memory, which is best scheduled on a node with fast disks. The scheduler will use this information to filter all nodes that meet these requirements. If multiple nodes meet the demand on average, Kubernetes will schedule pods on the node with the smallest number of pods. This is also the default behavior if the user does not specify any further requirements.

The desired state may not be established, for example, because the worker nodes do not have enough resources to run the application. In this case, the scheduler will retry to find an appropriate node until the state can be established.

### 9. Other resources

#### 9.1 The History of Kubernetes and the Legacy of Borg

* [From Google to the world: The Kubernetes origin story](https://cloud.google.com/blog/products/containers-kubernetes/from-google-to-the-world-the-kubernetes-origin-story) , by Craig McLuckie (2016)
* [Large-scale cluster management at Google with Borg](https://research.google/pubs/pub43438/), by Abhishek Verma, Luis Pedrosa, Madhukar R. Korupolu, David Oppenheimer, Eric Tune,John Wilkes (2015)

#### 9.2 Kubernetes Architecture

* [Kubernetes Architecture explained | Kubernetes Tutorial 15](https://www.youtube.com/watch?v=umXEmn3cMWY)

#### 9.3 RBAC

* [Demystifying RBAC in Kubernetes](https://www.cncf.io/blog/2018/08/01/demystifying-rbac-in-kubernetes/), by Kaitlyn Barnard

#### 9.4 Container Runtime Interface

* [Introducing Container Runtime Interface (CRI) in Kubernetes (2016)](https://kubernetes.io/blog/2016/12/container-runtime-interface-cri-in-kubernetes/)

#### 9.5 Kubernetes networking and CNI

* [What is Kubernetes networking?](https://www.vmware.com/topics/glossary/content/kubernetes-networking.html)

#### 9.6 Internals of Kubernetes Scheduling

* [A Deep Dive into Kubernetes Scheduling](https://thenewstack.io/a-deep-dive-into-kubernetes-scheduling/), by Ron Sobol (2020)

#### 9.7 Kubernetes Security Tools

* [Popeye](https://github.com/derailed/popeye)
* [kubeaudit](https://github.com/Shopify/kubeaudit)
* [kube-bench](https://github.com/aquasecurity/kube-bench)

#### 9.8 Kubernetes Playground

* [Play with Kubernetes](https://labs.play-with-k8s.com)

![Insert picture description here](https://img-blog.csdnimg.cn/282cb85495424bde9a8bd1bbdbc7b630.gif#pic\_center)
