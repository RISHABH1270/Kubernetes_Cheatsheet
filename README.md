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

## 🧩 Create a Pod on a Worker Node (Imperative and Declarative)

Imperative Way (One-liner Command)
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
  nodeSelector:
    kubernetes.io/hostname: multinode-cluster-m02
```
```bash
kubectl apply -f pod.yaml
```

