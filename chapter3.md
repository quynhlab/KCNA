# KCNA3: Container Orchestration

### 1 Overview

In this chapter, we'll look at the challenges and opportunities of container orchestration, and why it places special demands on networking and storage.

Before we start talking about container orchestration, let's take a few steps back and understand what a container is and what makes it so useful when building and operating applications.

### 2. Learning Objectives

By the end of this chapter, you should be able to:

* Explain what a container is and how it differs from a virtual machine.
* Describe the fundamentals of container orchestration.
* Discuss container networking and storage challenges.
* Explain the benefits of service mesh.

### 3. Using Containers

The history of application development is also the history of packaging those applications into different platforms and operating systems. Let's take an example of a simple web application written in the popular programming language Python. To run an application on a server or local machine, the system usually needs to meet the following specific requirements:

* Install and configure the base operating system
* Install the core Python packages to run the program
* Python extensions used by the installer
* Configure the network for your system
* Connect to third-party systems such as databases, caches or storage.

While developers know best about their applications and their dependencies, it is often the system administrators who provide the infrastructure, install all dependencies, and configure the system on which the application runs. This process is very error-prone and difficult to maintain, so the server is only configured for a single purpose, such as running a database or application server, and is then connected over a network.

To make more efficient use of server hardware, a virtual machine can be used to simulate a complete server with cpu, memory, storage, networking, operating system, and software. This allows running multiple isolated servers on the same hardware.

Before the widespread adoption of containers, server virtualization was the most efficient way to run standalone and tractable applications, but since the entire operating system, including the kernel, had to be run, it always brought some overhead.

Containers can be used to solve both of these problems, manage application dependencies, and run more efficiently than spinning up lots of virtual machines.

### 4. Container Basics

Contrary to popular belief, container technology is much older than one might expect. One of the earliest ancestors of modern container technology is the `chroot` command, which was introduced in Version 7 Unix in 1979. The chroot command can be used to isolate a process from the root filesystem and basically "hide" files from the process and simulate a new root directory. An isolated environment is a so-called chroot jail, in which a process cannot access files, but the files still exist on the system.

**Chroot directories can be created in different locations on the file system**

![](https://img-blog.csdnimg.cn/fd2667c19a654de7a955a1f6e03ddb20.png?x-oss-process=image/watermark,type\_ZHJvaWRzYW5zZmFsbGJhY2s,shadow\_50,text\_Q1NETiBAZ2hvc3R3cml0dGVu,size\_20,color\_FFFFFF,t\_70,g\_se,x\_16)

Although chroot is a fairly old technique, it is still used in some popular software projects. The container technology we have today still embodies this concept, but in a modernized version with a lot of features. To isolate processes better than chroot, current Linux kernels provide features like namespaces and cgroups. Namespaces are used to isolate various resources, such as the network. A complete abstraction of network interfaces and routing tables can be provided using network namespaces. This allows the process to have its own IP address. Linux Kernel 5.6 currently provides 8 namespaces:

* `pid` - Process ID provides a process with its own set of process IDs.
* `net` - network allows a process to have its own network stack, including IP addresses.
* `MNT` - mount abstract filesystem view and manage mount points.
* `Ipc` - Interprocess communication provides separation of named shared memory segments.
* `user` - Give the process its own set of user and group ids.
* `uts` - Unix time sharing allows processes to have their own hostname and domain name.
* `cgroup` - a new namespace that allows processes to have their own set of Cgroup roots.
* `time` - The latest namespace can be used to virtualize the system's clock.

cgroups are used to organize processes in hierarchical groups and allocate resources such as memory and CPU to them. When you want to limit the memory of your application container, let's say 4GB, cgroups are used to ensure these limits. Launched in 2013, Docker has become synonymous with building and running containers. While Docker didn't invent the technology for running containers, they brought together existing technologies in an intelligent way to make containers more user-friendly and convenient. At first glance, containers appear to be very similar to virtual machines, but understanding the differences between them is critical. While a virtual machine emulates a complete machine, including the operating system and kernel, containers share the host's kernel and, as stated, are just independent processes. Virtual machines come with overhead, such as boot time, size, or resource usage to run the operating system. Containers, on the other hand, are actually processes, like a browser you can launch on your machine, so they start faster and take up less space.

**Traditional Deployment vs Virtualized Deployment vs Container Deployment** In many cases it is not a question of using containers or virtual machines, but using both technologies to benefit from the efficiency of containers, but still The security benefits of using the greater isolation of virtual machines.

![](https://img-blog.csdnimg.cn/ee5ae54292044affabc2b28727d162da.png?)

### 5. Running the container

To run industry standard containers, you don't need to use Docker; you can just follow the OCI [runtime-spec](https://github.com/opencontainers/runtime-spec) specification. The Open Container Initiative also maintains a container runtime reference implementation called [runC](https://github.com/opencontainers/runc). This low-level runtime is used by various tools to start containers, including Docker itself. If you're a developer and know object-oriented programming, you can imagine the relationship between a container image and a running container, like a class, and an instantiation of that class. After installing Docker, you can start the container like this:

```bash
docker run nginx
```

`runtime-spec` is closely related to `image-spec`, we will discuss `image-spec` in the next chapter, because it describes how to unpack a container image and then manage the entire container life cycle, from creation of the container environment, to startup process, stop and delete it. For your local machine, there are plenty of alternatives to choose from, some of which are just for building images like [buildah](https://buildah.io) or [kaniko](https://github.com/GoogleContainerTools /kaniko), while others serve as complete replacements for Docker, such as [podman](https://podman.io). Podman provides a similar API to Docker as an alternative. In addition, it provides some additional features like running containers without root privileges, and the use of the Pod concept we will discover later.

### 6. Build the image

Why are containers called containers in the first place? The metaphor used here is for the use of containers standardized according to ISO 668. The standard format of a sea container allows it to be easily stacked on a container ship, and whatever is inside can be easily unloaded with a crane or loaded onto a truck. You'll see that many terms in the container and cloud-native world follow this nautical theme. Docker reuses all components to isolate processes, such as `namespaces` and `cgroup`, but the key to helping containers break through is the introduction of container images. Container images make containers portable and easy to reuse on a variety of systems. Docker describes a container image as follows: A Docker container image is a lightweight, self-contained, executable package that contains everything needed to run an application: code, runtime, system tools, system libraries, and settings. In 2015, the image format that [Docker](https://www.docker.com/resources/what-container) made it popular was donated to the newly formed Open Containers Initiative, also known as [OCI image- spec](https://github.com/opencontainers/image-spec), available on GitHub. An image consists of filesystem packages and metadata.

Container Images Images can be built by reading instructions from a build file called `Dockerfile`. These instructions are almost identical to the instructions used to install the application on the server. Here is an example of a Dockerfile that includes a Python script:

![](https://img-blog.csdnimg.cn/06e2fd227b1b4c45bd7a6b0ff53f27c0.png?)

```bash
# Every container image starts with a base image.
# This could be your favorite linux distribution
FROM ubuntu:20.04

# Run commands to add software and libraries to your image
# Here we install python3 and the pip package manager
RUN apt-get update && \
    apt-get -y install python3 python3-pip

# The copy command can be used to copy your code to the image
# Here we copy a script called "my-app.py" to the containers filesystem
COPY my-app.py /app/

# Defines the workdir in which the application runs
# From this point on everything will be executed in /app
WORKDIR /app

# The process that should be started when the container runs
# In this case we start our python app "my-app.py"
CMD ["python3","my-app.py"]
```

If you already have Docker installed on your machine, you can build the image with the following command:

```bash
docker build -t my-python-image -f Dockerfile
```

Use the `-t my-python-image` parameter to specify a name tag for the image, and `-f Dockerfile` to specify where the `Dockerfile` can be found. This enables developers to manage all of the application's dependencies and package it to run, rather than leaving that task to another person or team. To distribute these images, you can use the container registry. This is nothing more than a web server where you can upload and download pictures. Docker has built-in push and pull commands:

```bash
docker push my-registry.com/my-python-image
docker pull my-registry.com/my-python-image
```

* Demo: Building Container Images [https://github.com/docker/getting-started.git](https://github.com/docker/getting-started.git)

### 7. Container Security

It is important to understand that containers have different security requirements than virtual machines. Many people rely on the isolation properties of containers, but this can be very dangerous. When containers are started on a machine, they always share the same kernel, which poses a risk to the entire system if the container is allowed to call kernel functions (such as killing other processes or modifying the host network by creating routing rules). You can learn more about kernel capabilities in the [Docker documentation](https://docs.docker.com/engine/security/#linux-kernel-capabilities). One of the biggest security risks (not just in the container world) is executing a process with too many privileges, especially starting a process as root or administrator. Unfortunately, this problem has been overlooked a lot in the past, with many containers running as root. A fairly new attack surface introduced by containers is the use of public images. The two most popular public image registries are [Docker Hub](https://hub.docker.com) and [Quay](https://quay.io), and while they provide publicly accessible images, you It must be ensured that these images have not been modified to contain malware. Sysdig has a great blog post on [How to Avoid Massive Security Issues and Build Secure Container Images](https://sysdig.com/blog/dockerfile-best-practices/). In general, security is not something that can only be implemented at the container layer. This is an ongoing process that requires adjustments over time. The 4Cs of cloud-native security can roughly illustrate what layers need to be protected when using containers. Make sure to cover each layer as it effectively protects the inner layer. The Kubernetes documentation is a good starting point for understanding layers.

The 4C's of Cloud Native Security, retrieved from the [Kubernetes documentation](https://kubernetes.io/docs/concepts/security/overview/)

![](https://img-blog.csdnimg.cn/67927473b2784becb9ed02295fce1338.png?)

### 8. Container Orchestration

Running several containers on a local machine or on a single server is fairly easy, but the way containers are used brings new challenges about container operations. The efficiency of this concept leads to applications and services becoming smaller and smaller, and you will find that modern applications can be composed of many containers. Having a lot of loosely coupled, isolated and independent small containers is the basis of what is called a microservices architecture. These small containers are small pieces of self-contained business logic that are part of a larger application. If a large number of containers must be managed and deployed, a system will soon be needed to help manage those containers. Issues that need to be addressed include:

* Provide computing resources such as virtual machines that can run containers
* Schedule containers to the server in an efficient way
* Allocate resources such as CPU and memory to the container
* Manage the availability of containers and replace them when they fail
* Scale container if load increases
* Provide a network to connect containers together
* Provide storage if the container needs to persist data.

Container orchestration systems provide a way to build clusters of multiple servers and host containers on them. Most container orchestration systems consist of two parts: the control plane that manages the containers and the worker nodes that actually host the containers. There have been several systems available for orchestration over the years, but most are no longer relevant today and the industry has chosen Kubernetes as the standard system for orchestrating containers.

### 9. Container Networking

Microservice architecture relies heavily on network communication. Unlike monolithic applications, microservices implement an interface that can be invoked to make requests. For example, in an e-commerce application, you could have a service that responds to product listings. Network namespaces allow each container to have its own unique IP address, so multiple applications can open the same network port; for example, you can have multiple containerized web servers that all open port 8080. To make the application accessible from outside the host system, the container is able to map a port in the container to a port in the host system. To allow containers to communicate across hosts, we can use `overlay network`, placing them in a virtual network that spans host systems. This makes communication between containers very easy without the need for system administrators to configure complex networking and routing between the host and containers. Most overlay networks also need to deal with IP address management, which would be a lot of work if implemented manually. In this case, the overlay network manages which container gets which IP address, and how traffic flows to access individual containers.

Most modern container networking implementations are based on [Container Networking Interface (CNI)](https://github.com/containernetworking/cni). CNI is a standard that can be used to write or configure network plugins, and to easily exchange different plugins across various container orchestration platforms.

![](https://img-blog.csdnimg.cn/7ec0f41953aa4e4484e6c4728959f32a.png?)

### 10. Service Discovery & DNS

For a long time, server management in traditional data centers was manageable. Many system administrators even remember all the IP addresses of important systems they must use. The vast list of servers, their hostnames, IP addresses, and uses -- all maintained manually -- is a daily affair. In a container orchestration platform, things are much more complicated:

* Hundreds of containers with unique IP addresses
* Containers are deployed on different hosts, different data centers and even geographical locations
* Containers or services need DNS to communicate. It's almost impossible to use an IP address
* Information about the container must be deleted from the system when it is deleted.

The solution to this problem is automation. All information is placed in the Service Registry instead of a manually maintained list of servers (in this case, containers). Finding other services on the network and requesting information about them is called Service Discovery.

* Modern DNS servers with service APIs can be used to register new services when they are created. This method is very simple, as most organizations already have properly functioning DNS servers.
* Use a highly consistent data store, especially for storing information about the service. Many systems are capable of high-availability operation with robust failover mechanisms. Popular choices, especially for clusters, are [etcd](https://github.com/etcd-io/etcd), [Consul](https://www.consul.io) or [Apache Zookeeper] (https://zookeeper.apache.org).

### 11. Service Mesh

Because networking is such an important part of microservices and containers, networking can become very complex and opaque to developers and administrators. In addition to this, many functions such as monitoring, access control, or encryption of network traffic are required when containers communicate with each other. It is not necessary to implement all these functions in the application, just start a second container that implements these functions. The software used to manage network traffic is called a proxy. This is a server application that sits between the client and the server and can modify or filter network traffic before it reaches the server. Popular representatives are [nginx](https://www.nginx.com), [haproxy](http://www.haproxy.org) or [envoy](https://www.envoyproxy.io). Going a step further, the service mesh adds proxy servers to every container in the architecture. ![insert image description here](https://img-blog.csdnimg.cn/b73a535c57e2412697defe78f01441e3.png?) from [Istio.io](https://istio.io/v1.10/docs/ops/deployment /architecture/) can now use a proxy to handle network communication between services. Let's take encryption as an example. If two or more applications are supposed to encrypt their communications when they communicate with each other, then it is necessary to add libraries, configure and manage digital certificates to prove the identity of the applications involved. This can be a lot of work, and it can be error-prone if not very careful. When using a service mesh, applications do not communicate directly with each other, but instead route traffic through proxies. Currently the most popular service meshes are [istio](https://istio.io) and [linkerd](https://linkerd. io). Although they differ in implementation, the architecture is the same. The proxies in the service mesh form the data plane. This is where network rules are implemented and traffic flow is shaped. These rules are centrally managed in the control plane of the service mesh. Here you can define how traffic flows from service A to service B and what configuration should be applied to the proxy. So instead of writing code and installing libraries, just write a configuration file where you tell the service mesh that service A and service B should always communicate encrypted. The configuration is then uploaded to the control plane and distributed to the data plane to enforce the new rules. For a long time, the term "service mesh" only described the basic idea of ​​how traffic is handled using proxies in container platforms. The Service Mesh Interface ([Service Mesh Interface, SMI](https://smi-spec.io)) project aims to define a specification on how to implement a service mesh from different providers. They are very focused on Kubernetes, and their goal is to standardize the end-user experience for service meshes, as well as provide a standard for providers looking to integrate with Kubernetes. You can find the current [spec] on GitHub (https://github.com/servicemeshinterface/smi-spec).

### 12. Container Storage

From a storage perspective, containers have one glaring flaw: they are ephemeral. To understand exactly what it means, we need to understand what happens when a container starts with container images. Generally speaking, container images are read-only and consist of different layers that include everything you add during the build phase. This ensures that every time a container is launched from an image, you get the same behavior and functionality. As you can probably imagine, many applications require writing files. To allow writing to files, when you start a container from an image, a read-write layer is placed on top of the container image. ![Insert image description here](https://img-blog.csdnimg.cn/53f675035c1c4937b827fa778c065918.png?) Container layer, retrieved from [Docker documentation](https://docs.docker.com/storage/storagedriver /) The problem here is that this read-write layer is lost when the container is stopped or deleted. Just like your computer gets erased when you shut it down. To persist data, you need to write it to disk. If the container needs to persist data on the host, volumes can be used to achieve this. The concept and technique are very simple: instead of isolating the entire filesystem of a process, a directory residing on the host is passed into the container filesystem. If you think this will weaken the isolation of the container, you are right. When using container volumes, the host filesystem can be accessed efficiently. ![insert image description here](https://img-blog.csdnimg.cn/1e048697f5554e27af7f69424141c948.png?) Data is shared between two containers on the same host

When you orchestrate many containers, persisting data on the host where the containers are started may not be the only challenge. Often, data needs to be accessed by multiple containers launched on different host systems, or when a container is launched on different hosts, it can still access its volumes. Container orchestration systems like Kubernetes can help alleviate these issues, but a robust storage system attached to the host server is always required. ![insert image description here](https://img-blog.csdnimg.cn/813a4559c8c04efe84e276f7e32f2b4d.png?) Storage is provided through a central storage system. Containers on server A and server B can share a volume to read and write data

To keep up with the constant growth of various storage implementations, again, the solution is to implement a standard. [Container Storage Interface (CSI)](https://github.com/container-storage-interface/spec) provides a unified interface that allows attaching different storage systems, whether it is cloud storage or local storage.

### 13. Other resources

#### 13.1 History of containers

*["A Brief History of Containers: From the 1970s to the Present"](https://blog.aquasec.com/a-brief-history-of-containers-from-1970s-chroot-to-docker-2016), By Rani Osnat (2020)
* [Right here: Docker 1.0](https://web.archive.org/web/20160426102954/https://blog.docker.com/2014/06/its-here-docker-1-0/), Julian Barbier (2014)

#### 13.2 Chroot

* [chroot](https://wiki.ubuntuusers.de/chroot/)

#### 13.3 Container performance

* [Container Performance Analysis at DockerCon 2017](https://www.brendangregg.com/blog/2017-05-15/container-performance-analysis-dockercon-2017.html), by Brendan Gregg

#### 13.4 Best practices for how to build container images

* [Top 20 Dockerfile Best Practices](https://sysdig.com/blog/dockerfile-best-practices/), by Álvaro Iradier (2021)
* [3 simple tricks for smaller Docker images](https://learnk8s.io/blog/smaller-docker-images), by Daniele Polencic (2019)
* [Best practices for building containers](https://cloud.google.com/architecture/best-practices-for-building-containers)

#### 13.5 Alternatives to classic Dockerfile container builds

[Buildpacks vs Jib vs Dockerfile: A Comparison of Containerization Approaches](https://trainingportal.linuxfoundation.org/learn/course/kubernetes-and-cloud-native-essentials-lfs250/container-orchestration/%C3%81l), by James Ward (2020)

#### 13.6 Service Discovery

* [Service Discovery in a Microservices Architecture](https://www.nginx.com/blog/service-discovery-in-a-microservices-architecture/), by Chris Richardson (2015)

#### 13.7 Container Networking

* [Kubernetes Networking Part 1: Networking Essentials] (https://www.inovex.de/de/blog/kubernetes-networking-part-1-en/), By Simon Kurth (2021)
* [Life of a Packet](https://www.youtube.com/watch?v=0Omvgd7Hg1I) (I),by Michael Rubin (2017)
* [Computer Networking Introduction - Ethernet and IP (Heavily Illustrated)](https://iximiuz.com/en/posts/computer-networking-101/), by Ivan Velichko (2021)

#### 13.8 Container Storage

* [Managing Persistence for Docker Containers](https://thenewstack.io/methods-dealing-container-storage/), by Janakiram MSV (2016)

#### 13.9 Containers and kubernetes security

* [Secure containerized environments with updated thread matrix for Kubernetes](https://www.microsoft.com/security/blog/2021/03/23/secure-containerized-environments-with-updated-threat-matrix-for-kubernetes/), by Yossi Weizman (2021)

#### 13.10 Docker Container Playground

* [Play with Docker](https://labs.play-with-docker.com)
