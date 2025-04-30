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
kubectl describe pod my-pod
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


