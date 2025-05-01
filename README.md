# Kubernetes Hands-on with Minikube — Cheatsheet

This is a simple and beginner-friendly cheat sheet of commands I’m using while practicing Kubernetes locally with Minikube.
I will keep updating this with more commands and explanations — Coming Soon! 🚀

## 🚀 Start Minikube

Starts a single-node Minikube cluster locally.
```bash
minikube start
```
Starts a multi-node Minikube cluster (3 nodes) with a custom profile name multinode-cluster.
```bash
minikube start --nodes 3 -p multinode-cluster
```

## ⚙️ Managing Contexts (Clusters)

Lists all available contexts and highlights the current one with a *.
```bash
kubectl config get-contexts
```
Switches your active Kubernetes context to the multinode-cluster profile.
```bash
kubectl config use-context multinode-cluster
```

## 🖥️ Checking Cluster Nodes

Displays all nodes in the cluster and their statuses.
```bash
kubectl get nodes
```

## 📦 Get Namespaces

Get a list of namespaces in the cluster:
```bash
kubectl get namespaces
```

## 📋 Get all the resources in a Kubernetes cluster
```bash
kubectl get all
```
```bash
kubectl get all --all-namespaces
```

## 📚 To see the API documentation

```bash
kubectl explain pod
kubectl explain pod.spec
kubectl explain pod.metadata
kubectl explain pod.status
```

## 🧩 Create a Pod on a Worker Node (Imperative and Declarative)

Imperative Way (One-liner Command)
```bash
kubectl run nginx-pod --image=nginx:latest
```
```bash
kubectl run my-pod --image=nginx --overrides='
{
  "apiVersion": "v1",
  "spec": {
    "nodeSelector": {
      "kubernetes.io/hostname": "multinode-cluster-m02"
    }
  }
}' --restart=Never
```

📄 Declarative Way Using YAML - Create a file pod.yaml
```bash
vim pod.yaml
```
```bash
apiVersion: v1
kind: Pod
metadata:
  name: my-pod2
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
  nodeSelector:
    kubernetes.io/hostname: multinode-cluster-m02
```
```bash
kubectl create -f pod.yaml
kubectl apply -f pod.yaml
```

## 🔍 Describe a Pod

```bash
kubectl get pods
```
Now to check on whick worker node the pod is running:
```bash
kubectl get pods -o wide
```
Getting every information about a specific pods:
```bash
kubectl describe pod my-pod
```
Getting labels associated with a pod:
```bash
kubectl get pods my-pod2 --show-labels
```

## 🗑️ Delete a Pod

```bash
kubectl delete pod my-pod2
```

## 🛠️ Debugging a Pod

If the image within the container inside the pod has been altered or tampered:
```bash
  containers:
    - name: nginx
      image: nginx123
```
In this case, the pod won't be ready, and its status will show "ImagePullBackOff"
```bash
kubectl describe pod my-pod2
```
You'll find the latest error message in the Events section. 
           
Potential reasons:        
Registry unavailable         
Repository unavailable        
Image unavailable       
Authorization required       
Authorization failed      

To directly edit the configuration of an existing Resources in the cluster: 
```bash
kubectl edit pod my-pod2
```

## 🔐➡️ Get inside a Pod
```bash
kubectl exec -it my-pod2 -- sh
```
For multiple containers running inside a pod:
```bash
kubectl exec -it my-pod2 -c nginx -- bash
```
```bash
pwd
```

## ⚙️ Automatic creation of YAML
```bash
kubectl run nginx --image=nginx --dry-run=client
```
```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod-new.yaml
kubectl run nginx --image=nginx --dry-run=client -o json > pod-new.json
```   
It will create a yaml file and later you can make change in this as per your requirement:
```bash
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

## 📦📦📦 Kubernetes ReplicaSet 

ReplicationController are legacy one and with ReplicaSet you can target the existing pods with labels and selectors.         
Manages multiple replicas of an NGINX pod:
```bash
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
  labels:
    app: nginx
spec:
  replicas: 3  # Number of pod replicas
  selector:
    matchLabels:
      app: nginx
  template:  # Pod template
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 8080
```
```bash
kubectl apply -f rs.yaml
```
Get all the ReplicaSet:
```bash
kubectl get rs
```
Scale the ReplicaSet using Imperative way:
```bash
kubectl scale --replicas=5 nginx-replicaset
```

## 📦📦📦🛠️ Kubernetes Deployment:

Deployment --> ReplicaSet --> Pods    
         
Supports:      
Rolling updates (e.g., nginx:1.1 → nginx:1.2)    
Zero downtime       
Rollback if update fails      
Automates safe transitions between versions.      
ReplicaSet = raw pod management      
Deployment = smart version-controlled lifecycle manager   
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest  # ⬅️ Just change version here
        ports:
        - containerPort: 8080
```
One new pod with 1.2 is created (maxSurge: 1)           
One old pod with 1.1 is removed (maxUnavailable: 1)

```bash
kubectl apply -f dp.yaml
```

## 📜 Check Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

## 🔁 Rollback a Deployment

```bash
kubectl rollout undo deployment/nginx-deployment

```
