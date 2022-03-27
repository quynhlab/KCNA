# KCNA Exam Chapter 6: Continuous Delivery

### 1. Introduction<a href="#1__4" id="1__4"></a>

Deploying applications on any platform has come a long way over the years. In the beginning, applications may execute on the same machine they were written, later via physical media (floppy disk, USB stick, CD), now we check the server, build and application in the code, put it in a container, directly Deploy it to a platform like Kubernetes. \
The way we deliver applications is heavily influenced by the DevOps movement, which saw its breakthrough in the late 2000s. The DevOps movement is a cultural change that brings many new approaches

### 2. Learning Objectives <a href="#2__8" id="2__8"></a>

By the end of this chapter, you should be able to:

1. Discuss the importance of automation in integrating and delivering applications.
2. Understand the requirements for Git and version control systems.
3. Explain what a CI/CD pipeline is.
4. Discuss the concept of Infrastructure as Code (IaS).
5. Discuss the principles of GitOps and how it integrates with Kubernetes.

### 3. Application Delivery <a href="#3__17" id="3__17"></a>

The life cycle of every application starts with the code written. The source code is not only the basis of the application, but also the intellectual property and therefore the capital of most companies or individuals. We discovered a long time ago that the best way to manage source code is a version control system.

In 2005, Linus Torvalds created Git, the standard version control system used by almost everyone today. Git is a decentralized system that can be used to track changes in source code. Essentially, Git can use a copy of the code (called in a branch or branch) before your changes are merged back into the master branch.

Be sure to check out this web page to learn more about [git](https://git-scm.com) as it is a powerful industry standard tool used by almost all developers and administrators every day . \
After inspecting the source code, the next step before delivering the application is to build it, which may also include the building of container images, as described in the container orchestration chapter. \
To ensure the high quality of your app, the next step should be to test the app extensively and automatically to make sure that all functionality still works after someone makes changes.

The final step is to deliver the application to the platform it is supposed to run on. If your target platform is Kubernetes, then you can write a YAML file to deploy your application, and at the same time your newly built container image can be pushed to the container registry, where Kubernetes will download it for you.

Today, source code is not the only thing managed in a version control system. In order to fully utilize cloud resources, the principle of Infrastructure as Code (IaC) has become popular. Instead of installing the infrastructure manually, you can describe the infrastructure in a file and use the cloud provider's API to set it up. This allows developers to be more involved in setting up the infrastructure.

### 4. CI/CD <a href="#4_ci__cd_30" id="4_ci__cd_30"></a>

As services get smaller and deployments become more frequent, a logical and important step is the automation of the deployment process. The DevOps movement underscores the importance of frequent and rapid deployments. In a traditional setup, deployment would involve developers and administrators, many manual steps that are prone to error, and the constant fear that something will break.

Automation is the key to overcoming these obstacles, and today we know and use the principles of Continuous Integration/Continuous Delivery (CI/CD), which describe the different steps of application deployment, configuration, and even infrastructure.

* Continuous integration is the first part of the process, which describes the permanent build and test of written code. The use of high automation and version control allows multiple developers and teams to work on the same codebase.
* Continuous Delivery is the second part of the process that automates the deployment of pre-built software. In a cloud environment, you will often see software deployed to a development or delivery environment before being released and delivered to production systems.

To automate the whole workflow, you can use a CI/CD pipeline, which is really nothing more than a scripted form of all the relevant steps, running on a server or even in a container. Pipelines should integrate with a version control system that manages changes to the codebase. Whenever your code has a new revision ready to deploy, the pipeline starts executing scripts that build the code, run tests, deploy them to servers, and even perform security and compliance checks. \
In addition to generic scripting of pipeline steps, modern CI/CD tools have more capabilities, such as direct interaction and feedback from systems like Kubernetes.

Popular CI/CD tools include:

* [Spinnaker](https://spinnaker.io)
* [GitLab](https://about.gitlab.com)
* [Jenkins](https://www.jenkins.io)
* [Jenkins X](https://jenkins-x.io)
* [Tekton CD](https://github.com/tektoncd/pipeline)
* [Argo.](https://argoproj.github.io)

For a deeper understanding of DevOps, Site Reliability Engineering, and Infrastructure as Code, we strongly recommend that you study DevOps and [Introduction to Site Reliability Engineering (LFS162)](https://training.linuxfoundation.org/training/introduction-to -devops), a free course on edX.

### 5. GitOps <a href="#5_gitops_52" id="5_gitops_52"></a>

Infrastructure as code is a real revolution in improving the quality and speed of delivering infrastructure, and it works so well that today, configuration, networking, policy or security can all be described as code, even often with The software is in the same repository.

GitOps further takes the concept of Git as a single source of truth and integrates the configuration and change process of infrastructure with version control operations. \
If the code is branched and should be merged back into the master branch, you can create a merge or pull request that other developers can review before actually merging. This has been a best practice in software development for a long time, and also includes running a CI pipeline for every change that should be made. In GitOps, these merge requests are used to manage infrastructure changes.

The CI/CD pipeline has two different ways to achieve the changes you want:

* Push-based\
  A pipeline starts and runs tools that make changes in the platform. Changes can be triggered by commits or merge requests.
*Pull-based\
  The agent monitors the git repository for changes and compares the definitions in the repository with the actual running state. If changes are detected, the agent applies the changes to the infrastructure.

Examples of two popular GitOps frameworks that use a pull-based approach are [Flux](https://fluxcd.io) and [ArgoCD](https://argo-cd.readthedocs.io/en/stable/). ArgoCD is implemented as a Kubernetes controller, while Flux is built with the GitOps toolkit, a set of APIs and controllers that can be used to extend Flux or even build a custom delivery platform. \
The ArgoCD architecture is a good overview of how GitOps is implemented.

[ArgoCD Architecture, retrieved from ArgoCD Documentation](https://argo-cd.readthedocs.io/en/stable/operator-manual/architecture/)\
Kubernetes is particularly suitable for GitOps because it provides an API and was designed from the ground up for declarative resource configuration and changes. You may notice that Kubernetes uses an idea similar to the pull-based approach: monitoring the database for changes, and applying changes to the running state if they don't match the desired state. \
To learn more about GitOps in action and the use of ArgoCD and Flux, consider signing up for the [free course Introduction to GitOps (LFS169)](https://training.linuxfoundation.org/training/introduction-to-gitops-lfs169 /).

### 6. Other resources <a href="#6__73" id="6__73"></a>

10 Deploys Per Day - Start of the DevOps movement at Flickr

* [Velocity 09: John Allspaw and Paul Hammond, “10+ Deploys Per Day”](https://www.youtube.com/watch?v=LdOe18KhtT4)
* [10+ Deploys Per Day: Dev and Ops Cooperation at Flickr](https://www.slideshare.net/jallspaw/10-deploys-per-day-dev-and-ops-cooperation-at-flickr), by John Allspaw and Paul Hammond

Learn git in a playful way

* [Oh My Git! An open source game about learning Git!](https://ohmygit.org)
* [Learn Git Branching](https://learngitbranching.js.org)

Infrastructure as Code

* [Delivering Cloud Native Infrastructure as Code](https://www.pulumi.com/whitepapers/delivering-cloud-native-infrastructure-as-code/)
* [Unlocking the Cloud Operating Model: Provisioning](https://www.hashicorp.com/resources/unlocking-the-cloud-operating-model-provisioning)

Beginners guide to CI/CD

* [GitLab's guide to CI/CD for beginners](https://about.gitlab.com/blog/2020/07/06/beginner-guide-ci-cd/)

***
