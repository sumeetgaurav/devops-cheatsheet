# Kubernetes Advanced — 26 September 2026

How pods keep their data: PV, PVC, access modes and StatefulSets.

## Where the class started

### Assignment: basic K8s practice

- Create a local cluster using kind
- Put a pod in a separate namespace
- Make the pod publicly available and scale it to 50 replicas

### Recap questions

- What is Kubernetes? When was it founded, who built it and who maintains it?
- Monolith vs microservices vs monorepo
- Why microservices need Kubernetes
- What, why and how do we learn K8s? The class is currently at "how".

Covered so far: the architecture and planes, core resources, namespaces, pods, deployments, services, secrets and config maps. The goal now is that when you create a pod, service or deployment, you route it properly, and when you make a pod, you give it a volume for its storage.

## The problem: a pod forgets everything

- An nginx pod is running. Nginx is a reverse proxy server that can host web apps and route to internal APIs. Say it holds a web page or some data.
- The pod crashes. Kubernetes self-heals and creates a new pod. This is the expected behavior.
- The data inside is gone. Pods are stateless by nature and do not maintain state.

**Docker parallel:** When a Docker container crashes we keep data by mapping a Docker volume. Kubernetes has the same idea, called the Persistent Volume.

## The idea: shared cluster storage

Imagine a cluster with 10 GB of shared storage. Two workloads, W1 and W2, are both in the cluster. W1 has 5 GB and W2 has 5 GB, so the total shared storage is 10 GB.

## PV and PVC: two objects, two scopes

|  | PV (Persistent Volume) | PVC (Persistent Volume Claim) |
|---|---|---|
| **What it is** | A resource that gives a cluster a particular amount of storage, so a pod or deployment can access it | A request for storage made by the user |
| **Scope** | Cluster-wide, has no namespace | Created inside a namespace |
| **Role** | The supply: storage taken from the cluster | The bridge that lets isolated resources reach cluster-wide storage |

**Why the claim is needed.** Deployments, services, config maps and secrets are isolated inside a namespace. A PV sits at the cluster level, so a namespaced resource cannot reach it directly. The PVC is the namespaced resource that claims from it.

**Worked example:** The cluster has a 5 GB PV. Your deployment needs 2 GB, so your PVC claims 2 GB and the remaining 3 GB stays available in the cluster.

A pod that needs storage goes through a PVC. A pod that does not need storage skips both and needs no PV.

### Demo from class

Practical demo on AWS, with the files in the class GitHub repo. The nginx pod is given persistent storage, so when the pod is killed, the replacement gets the storage back.

```bash
# log in to AWS, then check the cluster
kubectl get nodes

# pod.yml -> nginx pod that mounts the persistent storage
# pvc.yml -> the claim (nano pvc.yml)
```

In short: a PVC is a request for storage by the user.

## Access modes

When building a K8s cluster and using EBS, which access mode do you use? It depends on the compute you are using.

| Mode | Meaning | When to use |
|---|---|---|
| RWO | Read Write Once | With EBS (Elastic Block Store). EC2 instances generally have RWO. |
| RWM | Read Write Many | With a shared file system. |

## Next up: StatefulSets

Everything above works for stateless applications. The follow-up question is what happens with stateful ones.

- A StatefulSet is similar to a Deployment resource.
- The difference: in a StatefulSet, the state is preserved.
