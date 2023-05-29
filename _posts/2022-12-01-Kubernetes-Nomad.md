---
layout: post
title:  "Nomad & Kubernetes - David vs Goliath played out in container orchestration"
date:   2022-12-04 12:22:00
comments: True
categories: [Software, Containers]
excerpt_separator: "<!--more-->"
---

Kubernetes and Nomad are both container orchestration tools that enable the management of containers across multiple hosts. 

Kubernetes has a large, rapidly growing ecosystem of services, support and tools that are widely available. Kubernetes is an open source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a layered architecture with various components that provide features such as cluster management, scheduling, service discovery, monitoring, secrets management and more1. Kubernetes also relies on a dynamic ecosystem of various loosely-coupled components, such as container runtimes, storage providers, network plugins, ingress controllers, service meshes, etc., that extend its functionality and interoperability.

<!--more-->

Nomad has a smaller but growing ecosystem of services, support and tools that are mostly provided by HashiCorp or its partners. Nomad is a simple and lightweight container orchestrator that can manage containers and non-container workloads. It has a single binary architecture that requires no external services for coordination or storage. Nomad only focuses on cluster management and scheduling and is designed to compose with other HashiCorp tools like Consul for service discovery/service mesh and Vault for secret management. Nomad also supports various drivers and plugins that enable it to work with different workloads, such as Docker, Java, Qemu, etc. and different storage and network providers.

Here is a quick comparison of the two platforms:

- **Simplicity**: Nomad is a single binary that requires no external services for coordination or storage, while Kubernetes is a collection of more than a half-dozen interoperating services that require etcd for coordination and storage.
- **Flexibility**: Nomad can manage containers and non-container workloads, such as Java, IIS on Windows, Qemu, etc., while Kubernetes is primarily designed to manage containers.
- **Consistency**: Nomad can be deployed in local dev, production, on-prem, at the edge and in the cloud in a consistent manner, while Kubernetes has different distributions for different environments, such as minikube, kubeadm, k3s, etc., which may lead to inconsistency in capabilities, configuration and management.
- **Scalability**: Nomad claims to support clusters up to 10,000 nodes and 1 million allocations with sub-second scheduling latency², while Kubernetes supports clusters up to 5000 nodes and 300,000 total containers.

Ultimately, the best choice for you will depend on your specific needs and requirements. If you are looking for a mature, widely used platform with a large community and a lot of resources, then Kubernetes is a good option. If you are looking for a simpler, easier to use platform with more flexibility, then Nomad is a good option.

Here are some additional factors to consider when choosing between Kubernetes and Nomad:

- Your experience with container orchestration: If you are new to container orchestration, then Nomad may be a better choice because it is easier to learn and use.
- The type of applications you are deploying: If you are deploying a wide variety of workloads, then Nomad may be a better choice because it is more flexible.
- Your budget: If you are on a tight budget, then Nomad may be a better choice because it is less expensive.

If you are still not sure which platform to choose, then you can try both of them and see which one you prefer. Both Kubernetes and Nomad offer free trials, so you can test them out without any commitment.