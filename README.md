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

