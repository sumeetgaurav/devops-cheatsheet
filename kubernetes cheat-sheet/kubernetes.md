# Kubernetes Fundamentals

## Building Blocks

The basic building blocks are:

![Monolith vs microservices](image.png)

The rule of thumb from the diagram above: a simple app with one job is a **monolith**, and an app with many different moving parts is **microservices**. Microservices are generally deployed on Kubernetes.

## What Is Kubernetes?

Kubernetes manages many Docker containers (or other containers) so that you can ship software smoothly.

## Why Do We Need It?

- With Docker alone, managing 1 or 2 containers is easy, but 100 containers become very hard to manage.
- Microservices can grow rapidly, which raises the chances of crashes, traffic problems and management headaches.
- The app needs autoscaling: more copies when traffic rises, fewer when it drops.
- Kubernetes is a robust orchestration system that lets you create, schedule, delete and inspect containers.

## Servers and Clusters

A **server** is a machine that runs your containers, using Docker or alternatives such as Podman and containerd. Containers are basically Linux processes, and servers run those processes. Every server has resources: RAM, CPU and disk. A **cluster** is a group of one or many servers.

The diagram below shows how these pieces fit together.

![Servers and clusters](image-1.png)

## Key Terms

| Term | Definition |
|---|---|
| **Container** | A packaged app. Under the hood, it is just an isolated Linux process. |
| **Server** | A machine with RAM, CPU and disk that runs containers. Also called an instance, node, worker, host or VM. |
| **Cluster** | One or many servers grouped together. |
| **Orchestration** | Automatically managing many containers: creating, scheduling, deleting, inspecting and scaling them. |
| **Autoscaling** | Adding or removing containers automatically as traffic changes. |

## The Big Idea

> Docker runs containers, and Kubernetes manages hundreds of them across many servers so that microservices apps stay running, scalable and organised.
