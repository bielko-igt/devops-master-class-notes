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

> GKE uses a managed Kubernetes cluster, so the Master node is managed by Google and is not available within kubectl.

**The applications can continue working even when the master node goes down.**
- this is because the running applications do not interact with the master node at all
- understandably, you wouldn't be able to make changes to these applications in such a scenario, but they would continue running

Check statuses of components running on the Master Node:

```
kubectl get componentstatuses
```
- ```
  NAME                 STATUS    MESSAGE             ERROR
  controller-manager   Healthy   ok
  scheduler            Healthy   ok
  etcd-0               Healthy   {"health":"true"}
  etcd-1               Healthy   {"health":"true"}
  ```

# Google Cloud regions and zones

Google Cloud has available regions all over the globe.
- these are useful not only for latency, but also for ensuring that your applications don't go down if one region goes down - by distributing between regions
- they are also useful for legal requirements - some countries have different data laws

Within a region, there may be a number of different physically separate data centres - zones.
- in that case, even distributing your application between just different zones of a region may be sufficient to prevent your application going down due to a data centre going down

# Installing GCloud and Kubectl in your Terminal

- install GCloud CLI
- ```
  gcloud auth login
  ```
- install kubectl
- connect to GKE with command found in Google Cloud when clicking 'Connect' in your cluster

# Deployment history, rollouts and rollbacks

Rollout - new release

```
kubectl rollout history deployment hello-world-rest-api
```

- ```
  deployment.apps/hello-world-rest-api
  REVISION  CHANGE-CAUSE
  1         <none>
  ```
- CHANGE-CAUSE='\<none>' is not good
- ```
  kubectl set image deployment hello-world-rest-api hello-world-rest-api=in28min/hello-world-rest-api:0.0.3.RELEASE --record=true
  ```
  - the record parameter records the entire command as the change-cause
  - next output of rollout history:
    ```
    deployment.apps/hello-world-rest-api
    REVISION  CHANGE-CAUSE
    1         <none>
    2         kubectl.exe set image deployment hello-world-rest-api hello-world-rest-api=in28min/hello-world-rest-api:0.0.3.RELEASE --record=true
    ```

## Rollback and pause

```
kubectl rollout undo deployment hello-world-rest-api --to-revision=1
```
- The revision 1 gets moved over as revision 3 and is now the most recent:
  ```
  deployment.apps/hello-world-rest-api
  REVISION  CHANGE-CAUSE
  2         kubectl.exe set image deployment hello-world-rest-api hello-world-rest-api=in28min/hello-world-rest-api:0.0.3.RELEASE --record=true
  3         <none>
  ```

During a large deployment, if you notice a problem, you may want to pause the deployment:

```
kubectl rollout pause deployment hello-world-rest-api
```

## Check logs of a specific instance

```
kubectl get pods
```

```
kubectl logs <name of pod>
```

Following (tailing) the logs:
```
kubectl logs -f <name of pod>
```

# Kubernetes YAML

Just like docker-compose, we can create a YAML file specifying the desired state for deployment.

This yaml can be generated from the current state of the deployment (becoming the desired state in the YAML):

```
kubectl get deployment hello-world-rest-api -o yaml > deployment.yaml
```

In order to then apply any changes we make to this deployment.yaml, we use the following command:
```
kubectl apply -f deployment.yaml
```
- the YAML contains the name of the deployment, so this command automatically has context for which deployment to apply from the file

This way, however, the service we set up (that exposes a port on our pods) is not included in the yaml. We need to set this up separately:

```
kubectl get service hello-world-rest-api -o yaml > service.yaml
```

We then open up the deployment.yaml, and append the service.yaml to the end, separating the two with triple hyphens -> ---

```
...  
  observedGeneration: 7
  readyReplicas: 3
  replicas: 3
  updatedReplicas: 3
---
apiVersion: v1
kind: Service
metadata:
  annotations:
...
```

When we apply this edited deployment.yaml, the service gets set up as well.

Let's also remove unnecessary elements from the deployment.yaml:

```
metadata:
  annotations:                                              <-- delete
    deployment.kubernetes.io/revision: "3"                  <-- delete
  creationTimestamp: "2023-01-23T10:00:18Z"                 <-- delete
  generation: 7                                             <-- delete
  labels:           
    app: hello-world-rest-api           
  name: hello-world-rest-api            
  namespace: default            
  resourceVersion: "727822"                                 <-- delete
  uid: 9473c57d-3a1a-4027-be32-633bd328d451                 <-- delete
spec:           
  progressDeadlineSeconds: 600                              <-- delete
  replicas: 2
  revisionHistoryLimit: 10                                  <-- delete
  selector:
    matchLabels:
      app: hello-world-rest-api
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null                               <-- delete
      labels:
        app: hello-world-rest-api
    spec:
      containers:
      - image: in28min/hello-world-rest-api:0.0.2.RELEASE
        imagePullPolicy: IfNotPresent
        name: hello-world-rest-api
        resources: {}                                       <-- delete
        terminationMessagePath: /dev/termination-log        <-- delete
        terminationMessagePolicy: File                      <-- delete
      dnsPolicy: ClusterFirst                               <-- delete
      restartPolicy: Always
      schedulerName: default-scheduler                      <-- delete
      securityContext: {}                                   <-- delete
      terminationGracePeriodSeconds: 30
status:                                     <-- remove status entirely
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    cloud.google.com/neg: '{"ingress":true}'
  creationTimestamp: "2023-01-23T10:02:03Z"                 <-- delete
  finalizers:
  - service.kubernetes.io/load-balancer-cleanup
  labels:
    app: hello-world-rest-api
  name: hello-world-rest-api
  namespace: default
  resourceVersion: "20269"                                  <-- delete
  uid: 213ba404-b2dc-48a9-a450-da80543bbdbd                 <-- delete
spec:
  allocateLoadBalancerNodePorts: true
  clusterIP: 10.124.6.175                                   <-- delete
  clusterIPs:                                               <-- delete
  - 10.124.6.175                                            <-- delete
  externalTrafficPolicy: Cluster                            <-- delete
  internalTrafficPolicy: Cluster                            <-- delete
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 32063
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: hello-world-rest-api
  sessionAffinity: None
  type: LoadBalancer
status:                                     <-- remove status entirely
```
To do this entire deployment at once, we first need to delete the existing deployment:

```
kubectl delete all -l app=hello-world-rest-api
```
- deletes all resources labeled with the name of our app - all the pods, services, deployments and replicasets

## Repeating a request to the LoadBalancer

```
watch curl <url>
```
Or in PowerShell:
```
while (1) {curl http://34.116.213.111:8080/hello-world; sleep 2; echo ""}
```

## Checking difference between Deployment.yaml and cluster

```
kubectl diff -f deployment.yaml
```

## Reduce downtime

You may reduce downtime by setting an amount of time for the containers on your pods to get ready
 - only then do the original pods get removed
 - the result is a smooth transition between the old and new release

```yaml
spec:
  minReadySeconds: 45
```

## Distributing LoadBalancer between two deployments, or choosing one

- we make two deployments in the YAML and write labels -> version: v1, v2
- if the LoadBalancer service does not have the version label, it distributes between **both** deployments
- otherwise, it **only** works for the deployment with that version

# More interesting kubectl commands

Show all pods, not just default namespace:
```
kubectl get pods --all-namespaces
```
- there are many pods in the namespace kube-system (internal)


Filter:
```
kubectl get pods --all-namespaces -l app=hello-world-rest-api
```

Sort by - the paths to the attributes correspond to the structure of the YAML specification:

```
kubectl get services --all-namespaces --sort-by=.spec.type
```

- gets sorted by the service type

Print a ton of info about the cluster:
```
kubectl cluster-info dump
```

List of nodes with how much cpu and memory they take up:

```
kubectl top node
```

List of pods with how much cpu and memory they take up:

```
kubectl top pod
```

Shortcuts only if you get comfortable with all of kubectl:
```
kubectl get svc // services
kubectl get ev // events
kubectl get rs // replicaset
kubectl get ns // namespace
kubectl get no // nodes
kubectl get po // pods
```