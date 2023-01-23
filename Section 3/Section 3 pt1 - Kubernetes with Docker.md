# Introduction to deployment with Kubernetes (K8S)

- Kubernetes logo: Helmsman wheel

Example of deploying an application on Kubernetes:

```sh
kubectl create deployment hello-world-rest-api --image=in28min/hello-world-rest-api:0.0.1.RELEASE

kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080
```

Adding multiple instances is as simple as running:
```sh
kubectl scale deployment hello-world-rest-api --replicas=3
```

- if one of the instances goes down, it gets automatically replaced by another instance in a matter of seconds

Automatically scale the amount of instances depending on their load, maximising them at 70%, with 10 instances at most:

```sh
kubectl autoscale deployment hello-world-rest-api --max=10 --cpu-percent=70
```

# Kubernetes on Google Cloud
€281 free credit (375 if signed into Google with business account)

- enable Kubernetes engine
- you will be automatically led to GKE (Kubernetes Engine) page - if not, search GKE in the website's search bar
- we will be running applications (**Workloads**) in **Clusters**

## Creating a Cluster

- Clusters -> Create
- Options:
   - Autopilot: Google manages your cluster (Recommended) - pay-per-pod
   - Standard: You manage your cluster - pay-per-node
- we will be choosing Standard in order to learn how to manage the number of nodes, etc.

# Connecting to the cluster

- Open Google Cloud Shell
  
```
Welcome to Cloud Shell! Type "help" to get started.
Your Cloud Platform project in this session is set to cool-eye-375315.
Use “gcloud config set project [PROJECT_ID]” to change to a different project.
```
- in our Cluster, click Connect
- copy the command shown on the website into Google Cloud Shell (example below)

  ```
  Fetching cluster endpoint and auth data.
  kubeconfig entry generated for in28minutes-cluster.
  ```
- kubectl - Kube Controller - interacting with the cluster
  - Deploying an application, increasing number of instances, everything else shown at the top of this file.
  - kubectl version
- Deploying an application:
  ```
  kubectl create deployment hello-world-rest-api --image=in28min/hello-world-rest-api:0.0.1.RELEASE
  ```
    - ```
      deployment.apps/hello-world-rest-api created
      ```
- Exposing a service:
  ```
  kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080
  ```
    - ```
      service/hello-world-rest-api exposed
      ```
    - we can check Services & Ingress on Google Cloud to see if the service is there
    - the column "Endpoints" on the page shows the exposed address and port
  
# Kubernetes Concepts

The command 'kubectl get events' shows that there are a lot of events required for creating the cluster and creating just one deployment.

- we created and exposed a simple deployment
- in turn Kubernetes created for us:
  - a pod, a replica set, a deployment and a service

Getting the created pod:
```
kubectl get pods
```
- ```
  NAME                                    READY   STATUS    RESTARTS   AGE
  hello-world-rest-api-569c4879d9-6bm4f   1/1     Running   0          23m
  ```

Getting the created replica set:
```
kubectl get pods
```
- ```
  NAME                              DESIRED   CURRENT   READY   AGE
  hello-world-rest-api-569c4879d9   1         1         1       25m
  ```

Getting the created deployment:
```
kubectl get deployment
```
- ```
  NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
  hello-world-rest-api   1/1     1            1           27m
  ```

Getting the created service:
```
kubectl get service
```
- ```
  NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)          AGE
  hello-world-rest-api   LoadBalancer   10.124.6.175   34.118.75.149   8080:32063/TCP   25m
  kubernetes             ClusterIP      10.124.0.1     <none>          443/TCP          64m
  ```
  - the kubernetes service is an internal service


- "-o wide" provides slightly more information on all get types above

Kubernetes uses the **One Responsibility Principle**: once concept - one responsibility.

# Pods

A Kubernetes node can contain multiple pods.

A pod is the smallest deployable unit - a pod can contain multiple containers (it is a collection of containers), but you cannot deploy containers without a pod.

- all the containers present in a pod share resources, and can communicate over localhost

A pod's attributes:
- apiVersion
- kind - string representing the REST resource this pod represents
- metadata
- spec - specification of desired behavior
- status - last observed, not up to date

Getting all details of a pod:
```
kubectl get pods
kubectl describe pod <>
```
- get a list of pods to get the pod's name
- describe that pod
- ```
  Name:             hello-world-rest-api-569c4879d9-6bm4f
  Namespace:        default
  Priority:         0
  Service Account:  default
  Node:             gke-in28minutes-cluster-default-pool-3dc976ce-7lh2/10.186.0.3
  Start Time:       Mon, 23 Jan 2023 10:00:19 +0000
  Labels:           app=hello-world-rest-api

                    pod-template-hash=569c4879d9
  Annotations:      <none>
  Status:           Running
  IP:               10.120.0.8
  IPs:
    IP:           10.120.0.8
  Controlled By:  ReplicaSet/hello-world-rest-api-569c4879d9
  Containers:
    hello-world-rest-api:
      Container ID:   containerd://ec49fb9ccbd166412dcc5e7c8dc24eac91445a396b0a999c67ec6676bf4da10b

      Image:          in28min/hello-world-rest-api:0.0.1.RELEASE
      Image ID:       docker.io/in28min/hello-world-rest-api@sha256:00469c343814aabe56ad1034427f546d43bafaaa11208a1eb0720993743f72be

      Port:           <none>
      Host Port:      <none>
      State:          Running
        Started:      Mon, 23 Jan 2023 10:00:26 +0000
      Ready:          True
      Restart Count:  0
      Environment:    <none>
      Mounts:
        /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2m8sf (ro)
  Conditions:
    Type              Status
    Initialized       True
    Ready             True
    ContainersReady   True
    PodScheduled      True
  Volumes:
    kube-api-access-2m8sf:
      Type:                    Projected (a volume that contains injected data from multiple sources)
      TokenExpirationSeconds:  3607
      ConfigMapName:           kube-root-ca.crt
      ConfigMapOptional:       <nil>
      DownwardAPI:             true
  QoS Class:                   BestEffort
  Node-Selectors:              <none>
  Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                              node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
  Events:                      <none>
  ```

  - the pod runs in the default namespace
  - labels are important for tying up pod with replicaset or service

# ReplicaSets