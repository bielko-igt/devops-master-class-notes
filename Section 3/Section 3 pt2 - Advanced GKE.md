# Quick overview of GKE on Google Cloud Console

- in Workloads, after clicking our specific deployment, we can perform multiple actions, such as a rolling update and scaling
- we can inspect and edit the yaml of the workload

# Understanding Kubernetes Architecture - Node and Nodes

## Cluster:
- Master Node - manages cluster
- Worker Mode - regular application node with pods

## Master Node:

- API Server (kube-apiserver)
  - Google Cloud, kubectl, they all talk to the cluster through this API server
- Distributed Database (etcd)
  - all the details of the commands we are executing are being stored here
  - thanks to this, the etcd stores the desired state
- Scheduler (kube-scheduler)
  - schedules the pods onto the nodes
- Controller Manager (kube-controller-manager)
  - kubectl manager
  - ensures that the state of the cluster matches the desired state

## Worker Node:
- Node Agent (kubelet)
  - monitors actions on the node and communicates them back to the Master mode
- Networking Component (kube-proxy)
  - enables exposing the deployment as a service
- Container Runtime (**any** CRI - docker, rkt, etc. - any container compatible with Open Container Interface)
  - to run the containers inside pods
- Pods (multiple pods running containers)
  - the applications we want to run are running on pods

**The applications can continue working even when the master node goes down.**
- this is because the running applications do not interact with the master node at all
- understandably, you wouldn't be able to make changes to these applications in such a scenario, but they would continue running

