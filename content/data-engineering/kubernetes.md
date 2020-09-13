# Kubernetes

- [ ] Break the monolith. 
- [ ] Decoupling app from infrastructure
- [ ] Imagine microservices architecture. How to communicate, distribute, manage, ...
- [ ] Ability to change components
- [ ] We need automation to deploy a large number of microservices
- [ ] NoOps development
- [ ] Sys admins take care of Kubernetes and the rest is automated.
- [ ] Kubernetes abstracts away the hardware infrastructure and exposes your whole datacenter as a single enormous computational resource
- [ ] Scaling at per service level
- [ ] Each component is developed and deployed independently of the others
- [ ] How to debug and trace -> Define sagas, traceability and saga level
- [ ] Use of distributed logging systems like Zipkin
- [ ] DevOps: organizations are realizing it’s better to have the same team that develops the application also take part 
    in deploying it and taking care of it over its whole lifetime. 
- [ ] NoOps:  you want the developers to deploy applications themselves without knowing anything about the hardware 
    infrastructure and without dealing with the ops team
- [ ] Build oyur own container from scratch https://www.youtube.com/watch?v=8fi7uSYlOdc
- [ ] Containers are much lightweight than VMs
- [ ] Containers thanks to Linux Namescapes and Linux Control Groups
- [ ] Docker main concepts: images, registries, containers
- [ ] To run an application on kubernetes first you have to create the container image
- [ ] You can modify number of copies or automate it.
- [ ] Because a containerized application already contains all it needs to run, the system administrators don’t need to install anything to
      deploy and run the app. On any node where Kubernetes is deployed, Kubernetes can run the app immediately without any help from the sysadmins. 
- [ ] all the nodes are now a single bunch of computational resources that are waiting for applications to consume them.
- [ ] an image contains many different layers. Layers can be shared
- [ ] Dev team is in charge of docker
- [ ] SYsteam in charge of Kubernetes
- [ ] Use kops tool to use kubernetes on AWS 

```shell script
 docker run busybox echo "Hello world"
```

```shell script
 docker run <image>:<tag>
```

Dockerfile
```shell script
FROM node:7
ADD app.js /app.js
ENTRYPOINT ["node", "app.js"]
```

Create an image called kubia from contents current directory
```shell script
 docker build -t kubia .
```

Run an image called kubia-container from an image called kubia. Map port and run in daemon mode
```shell script
 docker run --name kubia-container -p 8080:8080 -d kubia
```

List running containers
```shell script
docker ps
```

Get additional info
```shell script
docker inspect kubia-container
```

Execute shell
```shell script
 docker exec -it kubia-container bash
```

Stop container
```shell script
 docker stop kubia-container
```

Remove container
```shell script
 docker rm kubia-container
```

Tagging image under additional tag
```shell script
 docker tag kubia luksa/kubia
```

Push image to docker hub
```shell script
 docker push luksa/kubia
```

Running image from tag
```shell script
 docker run -p 8080:8080 -d luksa/kubia
```

Get nodes
```shell script
kubectl get nodes
```

Deploy kubernete from command line
```shell script
 kubectl run kubia --image=luksa/kubia --port=8080 --generator=run/v1
```

- [ ] Kubernetes doesn't deal with containers drectly. It deals with Pods.
- [ ]  A pod is a group of one or more tightly related containers that will always run
      together on the same worker node and in the same Linux namespace
- [ ] A worker node contains many pods. A pod contains many containers.


List pods
```shell script
 kubectl get pods
```

Describe pod
```shell script
kubectl describe pod
```

- [ ] To make the pod accessible from the outside, you’ll expose it through a
      Service object
```shell script
 kubectl expose rc kubia --type=LoadBalancer --name kubia-http
```
rc is for replication controller

List services
```shell script
 kubectl get services
```

- [ ] Pods are ephemeral. A missing pod is replaced by a new one.
- [ ] A service has a static IP

Increase replica count
```shell script
kubectl scale rc kubia --replicas=3
```
I detail number of replicas that I want, not how many to add. Declarative and not imperative

Access to dashboard
```shell script
 kubectl cluster-info | grep dashboard
```

- [ ] All containers in a pod are in the same working node. A pod is not distributed among different nodes
- [ ] Pods allow related containers to work together
- [ ] All containers in a node share same hostname and network adresses
- [ ] Filesystems are isolated in a pod
- [ ] They can share files through volumes
- [ ] All pods in a Kubernetes cluster reside in a single flat, shared, network-address space
- [ ] A pod is the basic unit of sccaling
- [ ] A pod shouldn't contain multiple containers if they can run in multiple machines
- [ ] Yet yaml of deployed pod
```shell script
 kubectl get po kubia-zxzij -o yaml
```

- [ ] Basic pod manifest
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: kubia-manual
spec:
 containers:
 - image: luksa/kubia
 name: kubia
 ports:
 - containerPort: 8080
 protocol: TCP
```

- [ ] Create pod from yaml
```shell script
 kubectl create -f kubia-manual.yaml
```
- [ ] Container logs to standard output
```shell script
 docker logs <container id>
 kubectl logs kubia-manual
```

- [ ] When a pod is deleted logs are deleted too
- [ ] You can setup a centralized logging store
- [ ] Forward local port for debugging purposes
```shell script
 kubectl port-forward kubia-manual 8888:8080
 curl localhost:8888
```

- [ ] App label describes microservice
- [ ] Rel label: stable, beta, canary
- [ ] Canary release is when you release version to a small fraction of users to test it.
- [ ] Show labels
```shell script
 kubectl get po --show-labels
```
- [ ] Create labels
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: kubia-manual-v2
labels:
 creation_method: manual
 env: prod
spec:
 containers:
 - image: luksa/kubia
 name: kubia
 ports:
 - containerPort: 8080
 protocol: TCP
```

- [ ] Modify labels
```shell script
 kubectl label po kubia-manual creation_method=manual
```

- [ ] Filter pod by label
```shell script
 kubectl get po -l creation_method=manual
```

- [ ] Categorize nodes with labels
```shell script
 kubectl label node gke-kubia-85f6-node-0rrx gpu=true
```

- [ ] List nodes by labels
```shell script
 kubectl get nodes -l gpu=true
```

- [ ] Schedule pod by worker label
```ỳaml
apiVersion: v1
kind: Pod
metadata:
 name: kubia-gpu
spec:
 nodeSelector:
 gpu: "true"
 containers:
 - image: luksa/kubia
 name: kubia
```

- [ ] get namespaces
```shell script
 kubectl get ns
```

- [ ] FIlter by namespaces
```shell script
 kubectl get po --namespace kube-system
```

- [ ] Creating namespace from yaml
```shell script
apiVersion: v1
kind: Namespace
metadata:
 name: custom-namespace


 kubectl create -f custom-namespace.yaml

-- or just
 kubectl create namespace custom-namespace
```

- [ ] Create in namespace
```shell script
 kubectl create -f kubia-manual.yaml -n custom-namespace
```

- [ ] Delete by name
```shell script
 kubectl delete po kubia-gpu

-- delete many
 kubectl delete po pod1 pod2
```

- [ ] Delete by label
```shell script
 kubectl delete po -l creation_method=manual
```

- [ ] Delete all in namespace
```shell script
 kubectl delete ns custom-namespace
```

- [ ] Delete pods in current namespace
```shell script
 kubectl delete po --all
```
Pods are recreated

- [ ] Remove almost all resources
```shell script
 kubectl delete all --all
```

- [ ] liveness probes: to check container is alive














             