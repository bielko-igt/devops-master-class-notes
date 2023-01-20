# Introduction to deployment with Kubernetes

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