# Dive into Kubernetes with Minikube: Your Ultimate Hands-On Cheatsheet!

This is a concise, beginner-oriented reference guide containing essential Kubernetes commands I’m using while practicing locally with Minikube. I will continue to expand it with additional commands and detailed explanations — updates coming soon! 🚀

Kubernetes takes care of all the concerns like container networking, resource management, security, high availability, fault tolerance, service discovery, scalability, load balancing, and orchestration—providing an enterprise-level solution for managing containerized applications at scale.

In Kubernetes, the cluster architecture is divided into two main parts:

### 🧠 Master Node (Control Plane)
The master node manages the cluster—it makes global decisions, like scheduling and scaling.

Key Components:      
kube-apiserver - The front-end of the control plane; receives all REST requests from users, CLI, or controllers.     
etcd - A key-value store for all cluster data; acts as the source of truth.     
kube-scheduler - Watches for unscheduled pods and assigns them to suitable worker nodes based on resources, policies, etc.     
kube-controller-manager - Runs controllers (like node, replication, endpoints) that ensure the cluster’s desired state is maintained.    
cloud-controller-manager - Handles cloud-specific logic (e.g., provisioning load balancers, managing storage in AWS, GCP, etc.).     

### ⚙️ Worker Node
The worker node actually runs the containerized applications.

Key Components:      
kubelet - Agent that runs on each worker node; ensures containers are running as expected by talking to the control plane.      
kube-proxy - Manages networking rules (NAT, IP forwarding) to allow communication between pods and services.         
Container Runtime - Software that actually runs the containers, such as containerd, Docker, or CRI-O.     
Pods - The smallest deployable units in Kubernetes, running your actual containers.     

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

Access the Kubernetes dashboard running within the minikube cluster:
```bash
minikube dashboard
```

Before going further, make sure you have installed kubectl, the command-line tool for interacting with Kubernetes clusters.
If not, install it from the official documentation here:     
   
👉 Install kubectl - https://kubernetes.io/docs/tasks/tools/

## ⚙️ Managing Contexts (Clusters)

Lists all available contexts and manage access to multiple clusters and highlights the current one with a *.
```bash
kubectl config get-contexts
```

Switches your active Kubernetes context to the minikube.
```bash
kubectl config use-context minikube
```

Stop your local cluster:
```bash
minikube stop
```

Delete your local cluster:
```bash
minikube delete
```

Delete all local clusters and profiles
```bash
minikube delete --all
```

## 🖥️ Get all the Nodes in the cluster

Displays all nodes in the cluster and their status.
```bash
kubectl get nodes
```

## 📦 Get Namespaces

Get a list of namespaces in the cluster:
```bash
kubectl get namespaces
```

## 📋 Get Resources in a Kubernetes cluster

Get all resources in the default namespace:
```bash
kubectl get all
```

Get all resources in a specific namespace (e.g., kube-system):
```bash
kubectl get all -n kube-system
```

Get all resources in all namespaces:
```bash
kubectl get all -A
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
```
```bash
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
```yaml
  containers:
    - name: nginx
      image: nginx123
```
In this case, the pod won't be ready, and its status will show "ImagePullBackOff" or "ErrImage"
```bash
kubectl describe pod my-pod
```
You'll find the latest error message in the Events section. 
           
- Potential reasons:        
- Registry unavailable         
- Repository unavailable        
- Image unavailable       
- Authorization required       
- Authorization failed      

To directly edit the configuration of an existing Resources in the cluster: 
```bash
kubectl edit pod my-pod
```

## 🔐➡️ Get inside a Pod

Accessing a Pod's shell is similar to how you exec into a Docker container.     

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

Kubernetes supports both YAML and JSON for defining configuration files for Pods, Deployments, Services, etc.       
```bash
kubectl run nginx --image=nginx --dry-run=client
```

Redirects the command output and saves it into a new file - 
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

ReplicationController is a legacy Kubernetes resource. The modern and recommended alternative is ReplicaSet by this you can target and manage the existing pods with labels and selectors.     

- A ReplicaSet ensures that the desired number of Pods are always running.    
- It monitors the health of each Pod continuously.    
- When handling traffic, it uses load balancing to route requests to healthy Pods, ensuring high availability.     

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
  selector:    # ReplicaSet
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

Pods → ReplicationController → ReplicaSet → Deployment      
         
Key Capabilities with Deployments:

- Seamless rolling updates (e.g., nginx:1.1 → nginx:1.2)
- Zero downtime during updates
- Automatic rollback on failure
- Safely automates version transitions
- Users remain unaffected as traffic is directed only to active and healthy Pods.      

Breakdown:                
       
ReplicaSet = Low-level Pod manager    
Deployment = High-level, version-aware lifecycle orchestrator        
      
A Deployment in Kubernetes creates and manages ReplicaSets under the hood.      
                  
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
  strategy:       # Deployment
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
One new pod with nginx version 1.2 is created (maxSurge: 1)           
One old pod with nginx version 1.1 is removed (maxUnavailable: 1)

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
A Service selects pods using labels and selectors then forwards traffic to them.     
        
Pod-to-Pod Communication? → Yes, typically via a Service      
While Pods can technically communicate directly using their IPs, this is not recommended because Pod IPs are ephemeral — they change when a Pod restarts or reschedules. Instead, use a Service to give a stable DNS name and virtual IP.       
Example: One Pod calls another via http://my-service.namespace.svc.cluster.local
          
There are 4 types of Services:     

### 🔌 NodePort (Access the application through the port exposed by the node and then internal routed to targeted port of the application)
  
Sample YAML for Nodeport      

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport-service
spec:
  type: NodePort
  selector:
    app: nginx   # This must match labels in your Pod/Deployment
  ports:
    - protocol: TCP
      nodePort: 30080     # External port exposed on each node (30000-32767)
      port: 80            # Port exposed by the service inside the cluster
      targetPort: 80      # Port on the pod/container  
```

Get the Node IP(s):
```bash
kubectl get nodes -o wide
```

🧭 Step-by-Step Traffic Flow -    

- Client Initiates Request - A user (browser, curl, etc.) makes a request to any Kubernetes node's external IP on port 30080.        
```bash
http://<node-ip>:30080
```
- NodePort Receives the Request - The Kubernetes NodePort service listens on port 30080 on every worker node, even if the Pod isn't running there.
- Service Layer Forwards the Request - The request is routed internally to port 80 of the Service, which is a stable virtual IP inside the cluster.
- Service Uses Selector to Find Pods - The Service checks for all Pods with the label is app: nginx
- Load-Balancing to Pods - The Service load-balances traffic across matching Pods and sends it to the Pod’s targetPort (80), where nginx is running.
- Pod Handles the Request - The nginx container listens on port 80, processes the request, and sends back a response.

### 🌀 ClusterIP (For Internal access)

A frontend connects to a backend through the cluster network, typically using a ClusterIP Service.

- Frontend Pods: Your UI or client-side logic
- Backend Pods: API, database access, business logic
- ClusterIP Service: Exposes the backend internally

Sample YAML for ClusterIP

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP               # Default type, exposes service inside the cluster only
  selector:
    app: backend                # This must match labels in your backend Pod/Deployment
  ports:
    - protocol: TCP
      port: 80                  # Port exposed by the service (used by frontend to connect)
      targetPort: 8080          # Port where the backend container is actually listening
```

Frontend Pod connects using the service name:      
fetch("http://backend-service/")     

Get the services:
```bash
kubectl get svc
```

Describe the services:
```bash
kubectl describe svc backend-service
```

#### Endpoints :- An endpoint is the IP address of the Pod that the Service routes traffic to. Every Pod in Kubernetes is assigned a private IP address within the cluster. However, these IPs are ephemeral — they can change if the Pod is restarted, rescheduled, or replaced and Exposing Pod IPs directly to users is not safe and can pose a security risk.

### 📥 LoadBalancer (To access the application on a domain name or Public IP address without using the port number)

When you want your application to be accessible from the internet, use a LoadBalancer Service. Once created, your cloud provider will provision a public IP (external IP).      

A LoadBalancer Service still uses internal endpoints (Pod IPs) to route traffic behind the scenes. These Pod IPs are ephemeral — they can change when Pods are recreated. That's why Kubernetes uses Services as a stable access point, keeping direct Pod IPs hidden from users for security and reliability.      

Sample YAML for LoadBalancer:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-loadbalancer
spec:
  type: LoadBalancer              # Exposes the service externally via cloud load balancer
  selector:
    app: frontend                 # Must match labels on frontend Pods
  ports:
    - protocol: TCP
      port: 80                    # External port
      targetPort: 3000            # Port where the frontend container is actually listening
```

Check the service:
```bash
kubectl get svc
```

Access the app via:
```bash
http://<EXTERNAL-IP>/<port>
```

### ↗️ External (To use an external DNS for routing)

Use this when you want your Pods to access an external service using a Kubernetes service name, and let DNS handle the actual redirection.

Sample YAML for External:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

 A Pod in the prod namespace queries my-service, Kubernetes returns a CNAME pointing to my.database.example.com, and the Pod connects to the external host via DNS.

 ## Namespaces

Namespaces provide an additional layer of isolation to help organize and separate Kubernetes objects. If no namespace is specified, the object is created in the default namespace. Kubernetes system components and internal objects are created in the kube-system namespace.      

Resources within the same namespace can communicate with each other, but they cannot directly access objects in a different namespace unless explicitly configured to do so (FQDN).
 

