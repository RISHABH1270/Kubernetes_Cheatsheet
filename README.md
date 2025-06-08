# Dive into Kubernetes with Minikube: Your Ultimate Hands-On Cheatsheet!

This is a concise, beginner-oriented reference guide containing essential Kubernetes commands I’m using while practicing locally with Minikube. I will continue to expand it with additional commands and detailed explanations — updates coming soon! 🚀

## 🚀 Start Minikube      

Minikube is a local Kubernetes environment designed to simplify learning and development. It allows you to run a Kubernetes cluster on your local machine using Docker or a compatible container runtime.
        
To install the latest minikube stable release on x86-64 macOS using Homebrew:       
```bash
brew install minikube
```

For more details, refer to the https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Farm64%2Fstable%2Fbinary+download

Starts a single-node Minikube cluster locally.
```bash
minikube start
```
Starts a multi-node Minikube cluster (3 nodes) with a custom profile name multinode-cluster.
```bash
minikube start --nodes 3 -p multinode-cluster
```

## ⚙️ Managing Contexts (Clusters)

Lists all available contexts and manage access to multiple clusters and highlights the current one with a *.
```bash
kubectl config get-contexts
```
Switches your active Kubernetes context to the multinode-cluster profile.
```bash
kubectl config use-context multinode-cluster
```

## 🖥️ Get all the Nodes in the cluster

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

A Pod is the smallest deployable unit in Kubernetes. It can consists one or more containers.       

Imperative Way (One-liner Command)
```bash
kubectl run nginx-pod --image=nginx:latest
```

Create pod on a specific node:

```yaml
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
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
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
kubectl get pods my-pod --show-labels
```

## 🗑️ Delete a Pod

```bash
kubectl delete pod my-pod
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
kubectl describe pod my-pod
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
kubectl edit pod my-pod
```

## 🔐➡️ Get inside a Pod
```bash
kubectl exec -it my-pod -- sh
```
For multiple containers running inside a pod:
```bash
kubectl exec -it my-pod -c nginx -- bash
```
```bash
pwd
```

## ⚙️ Automatic creation of YAML

Kubernetes supports both YAML and JSON for defining configuration files like Pods, Deployments, Services, etc.       
    
```bash
kubectl run nginx --image=nginx --dry-run=client
```
```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod-new.yaml
kubectl run nginx --image=nginx --dry-run=client -o json > pod-new.json
```   
It will create a yaml file and later you can make change in this as per your requirement:
```yaml
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
```yaml
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

Pods --> ReplicationController --> ReplicaSet --> Deployment       
         
Supports:      
Rolling updates (e.g., nginx:1.1 → nginx:1.2)    
Zero downtime       
Rollback if update fails      
Automates safe transitions between versions.      
ReplicaSet = raw pod management      
Deployment = smart version-controlled lifecycle manager       
              
```yaml
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

## 🌍 Service in Kubernetes

Services exposes your app to outer world and it provides you a consistent endpoint (IP + DNS name).      
A Service selects pods using labels and then forwards traffic to them.      
          
There are 4 types of Services:     

- NodePort (Access the application through the port exposed by the node and then internal routing to targeted port happens)
  
#### Sample YAML for Nodeport
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      nodePort: 30080     # External port exposed on each node (30000-32767)
      port: 80            # Port exposed by the service inside the cluster
      targetPort: 80      # Port on the pod/container  
```

- ClusterIP(For Internal access)
- LoadBalancer(To access the application on a domain name or IP address without using the port number)
- External (To use an external DNS for routing)

