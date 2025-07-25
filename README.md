# 🚀 Dive into Kubernetes with Minikube: Your Ultimate Hands-On Guide

Welcome to your **beginner-friendly reference guide** for practicing Kubernetes locally using **Minikube**. This cheatsheet is a growing collection of essential commands, architectural insights, and concepts that I’m learning and applying. Updates and deep dives coming soon! 🌱

---

## 📦 Why Kubernetes?

Kubernetes simplifies and supercharges container management by handling:

- ✅ **Container Networking**
- ✅ **Resource Management**
- ✅ **Security & Access Control**
- ✅ **High Availability & Fault Tolerance**
- ✅ **Service Discovery**
- ✅ **Auto-scaling & Load Balancing**
- ✅ **Orchestration at Scale**

It’s an **enterprise-grade solution** built for running containerized applications with confidence and control.

---

## 🧠 Kubernetes Cluster Architecture

A Kubernetes cluster has two primary components:

### 🎯 Control Plane (Master Node)

This is the brain of the cluster — it manages scheduling, decisions, and the overall cluster state.

**Key Components:**

- **`kube-apiserver`** – Handles all REST communication between users and the cluster.
- **`etcd`** – A distributed key-value store that acts as the single source of truth for all cluster data.
- **`kube-scheduler`** – Assigns new Pods to nodes based on available resources and constraints.
- **`kube-controller-manager`** – Runs controllers that ensure the cluster state matches the desired state.
- **`cloud-controller-manager`** – Manages cloud-specific logic (like storage, load balancers, etc.).

### ⚙️ Worker Nodes

These nodes run the actual containerized workloads — your apps live here.

**Key Components:**

- **`kubelet`** – An agent on each node that ensures containers are running as instructed by the control plane. Pod management at node level.
- **`kube-proxy`** – Manages pod networking, routing, and communication.
- **`Container Runtime`** – Runs containers (e.g., `containerd`, `CRI-O`, or `Docker`).
- **`Pods`** – The smallest deployable units in Kubernetes, containing your app containers.

---

## 🚀 Start Minikube      

Minikube is a local Kubernetes environment designed to simplify learning and development. It allows you to run a Kubernetes cluster on your local machine using Docker or a compatible container runtime.

To install the latest Minikube stable release on **x86-64 macOS** using Homebrew:

```bash
brew install minikube
```

For more details, refer to the https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Farm64%2Fstable%2Fbinary+download

🏁 Start a single-node Minikube cluster locally:
```bash
minikube start
```

Starts a multi-node Minikube cluster (3 nodes) with a custom profile name multinode-cluster:
```bash
minikube start --nodes 3 -p multinode-cluster
```

📊 Access the Kubernetes dashboard running within the minikube cluster:
```bash
minikube dashboard
```

Before going further, make sure you have installed kubectl, the command-line tool for interacting with Kubernetes clusters.
If not, install it from the official documentation here:     
   
👉 Install kubectl - https://kubernetes.io/docs/tasks/tools/

---

## ⚙️ Managing Contexts (Clusters)

🔍 Lists all available contexts and highlights the current one with a `*`:
```bash
kubectl config get-contexts
```

🎯 Switch to the Minikube context:
```bash
kubectl config use-context minikube
```

🛑 Stop your local cluster:
```bash
minikube stop
```

🧹 Delete your local cluster:
```bash
minikube delete
```

💣 Delete all local clusters and profiles:
```bash
minikube delete --all
```
---

## 📝 Set kube-config file as environment variable

This sets an environment variable called KUBECONFIG to point to your Kubernetes configuration file (by default located at $HOME/.kube/config).
```bash
export KUBECONFIG=$HOME/.kube/config
```

This file contains:
- cluster details (API server endpoint, cluster certificate)
- user credentials (certs or tokens)
- contexts (which cluster + user to use by default)

By exporting this variable, you tell kubectl (and other tools that use the Kubernetes client-go libraries) to use this configuration file for all subsequent commands.     

You might want to switch quickly between multiple clusters by pointing KUBECONFIG to different files.So by running you explicitly tell kubectl which config file to use.
```bash
export KUBECONFIG=/path/to/your/config
```

⚡ Example with multiple configs, Imagine you have:
- ~/.kube/config for your dev cluster
- ~/eks-config for your production EKS cluster
```bash
# Work on dev cluster
export KUBECONFIG=~/.kube/config
kubectl get nodes

# Now switch to EKS
export KUBECONFIG=~/eks-config
kubectl get nodes
```

---

## 🖥️ Get all the Nodes in the cluster

Displays all nodes in the cluster and their status:
```bash
kubectl get nodes
```
---

## 📦 Get Namespaces

Get a list of namespaces in the cluster:
```bash
kubectl get namespaces
```

---

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

---

## 📚 To see the API documentation

```bash
kubectl explain pod
kubectl explain pod.spec
kubectl explain pod.metadata
kubectl explain pod.status
```

---

## 🧩 Create a Pod on a Worker Node (Imperative & Declarative)

A **Pod** is the smallest deployable unit in Kubernetes — it can run one or more containers tightly coupled on the same host.

### ⚡ Imperative Way (One-liner Command)

Create a simple `nginx` pod:
```bash
kubectl run nginx-pod --image=nginx:latest
```

Create a pod on a specific node using --overrides:
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

📄 Declarative Way Using YAML:
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

---

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
---

## 🗑️ Delete a Pod

```bash
kubectl delete pod my-pod
```

---

## 🛠️ Debugging a Pod

Sometimes pods fail to start due to issues like incorrect images or pull errors. Here's how to diagnose and fix them like a pro 👨‍🔧    

### 🚫 Common Error: Broken Image

If the image name in the pod is incorrect or doesn't exist:
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
           
💡 Potential Causes
- 🔌 Registry unavailable
- 📦 Image or repository not found
- 🔒 Authentication required
- ❌ Authorization failed
- 📝 Typo in the image name

To directly edit the configuration of an existing Resources in the cluster: 
```bash
kubectl edit pod my-pod
```

---

## 🔐➡️ Get inside a Pod

Accessing a Pod's shell is similar to how you exec into a Docker container.     

```bash
kubectl exec -it my-pod -- sh
```

For multiple containers running inside a pod:
```bash
kubectl exec -it my-pod -c nginx -- bash
kubectl exec -it my-pod -c nginx -- bash
```

```bash
pwd
```

---

## ⚙️ Automatic creation of YAML

Kubernetes supports both YAML and JSON for defining configuration files for Pods, Deployments, Services, etc. 

```bash
kubectl run nginx --image=nginx --dry-run=client
```

Redirects the command output and saves it into a new file:
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
---

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

---

## 📦📦📦🛠️ Kubernetes Deployment:

**Pods → ReplicationController → ReplicaSet → Deployment**

From bare-bones containers to robust, self-healing infrastructure — this is the path to production-grade Kubernetes!
         
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

One new pod with nginx version 1.2 is created at a time(maxSurge: 1)           
One old pod with nginx version 1.1 is removed at a time(maxUnavailable: 1)

```bash
kubectl apply -f dp.yaml
```

---

## 📜 Check Rollout History

```bash
kubectl rollout history deployment/nginx-deployment
```

---

## 🔁 Rollout Restart

```bash
kubectl rollout restart deployment nginx-deployment
```

---

## 🔙 Rollback a Deployment

```bash
kubectl rollout undo deployment/nginx-deployment
```

---

## 🌍 Service in Kubernetes

Services exposes your app/pod to outer world and it provides you a consistent endpoint (IP + DNS name).  

🧠 Default DNS Name Format for Services by Kubernetes internal DNS system (kube-dns or CoreDNS):
```bash
<service-name>.<namespace>.svc.cluster.local
```

📌 What DNS names provide:
- Stable names for internal communication.
- No need to hard-code IPs (which can change).
- Auto-registration/removal as Services are created/destroyed.

A Service selects pods using labels and selectors then forwards traffic to them.        
        
Pod-to-Pod Communication? → Yes, typically via a Service      
    
While Pods can technically communicate directly using their IPs, this is not recommended because Pod IPs are ephemeral — they change when a Pod restarts or reschedules. Instead, use a Service to give a stable DNS name and virtual IP.       
Example: One Pod calls another via http://my-service.namespace.svc.cluster.local
          
### There are 4 types of Services:     

🔌 ***NodePort*** (Access the application through the port exposed by the node and then internal routed to targeted port of the application)
  
Sample YAML for Nodeport:
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

🌀 ***ClusterIP*** (For Internal access - default)

A frontend connects to a backend through the cluster network, typically using a ClusterIP Service.

- Frontend Pods: Your UI or client-side logic
- Backend Pods: API, database access, business logic
- ClusterIP Service: Exposes the backend internally

Sample YAML for ClusterIP:
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

- **Endpoints** :- An endpoint is the IP address of the Pod that the Service routes traffic to. Every Pod in Kubernetes is assigned a private IP address within the cluster. However, these IPs are ephemeral — they can change if the Pod is restarted, rescheduled, or replaced and Exposing Pod IPs directly to users is not safe and can pose a security risk.

📥 ***LoadBalancer*** (To access the application on a domain name or Public IP address without using the port number)

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

↗️ ***External*** (To use an external DNS for routing)

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

--- 

 ## 🛡️ Namespaces

Namespaces provide an additional layer of isolation to help organize and separate Kubernetes objects. If no namespace is specified, the object is created in the default namespace. Kubernetes system components and internal objects are created in the kube-system namespace.         

Resources within the same namespace can communicate using short names or direct Pod IPs. For cross-namespace access, objects must be referenced using their service fully qualified domain names (FQDN), as defined by the cluster’s internal DNS settings.          

Different permissions can be assigned to different namespaces using RBAC, allowing fine-grained access control across the cluster.         
 
Get a list of namespaces in the cluster:
```bash
kubectl get namespaces
```

Get all resources in a specific namespace (e.g., kube-system):
```bash
kubectl get all -n kube-system
```

Get all resources in the default namespace:
```bash
kubectl get all
```

Get all resources in all namespaces:
```bash
kubectl get all -A
```

Imperative Way (Direct CLI Command):
```bash
kubectl create namespace dev
```

Declarative Way (Using YAML Manifest):
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

```bash
kubectl apply -f namespace.yaml
```

⚠️ To delete a namespace in Kubernetes, This will delete all resources within the dev namespace, so use with caution.
```bash
kubectl delete namespace dev
```

To create a Pod in the dev namespace, Imperative Way:
```bash
kubectl run nginx-pod --image=nginx:latest --restart=Never -n dev
```

Declarative Way (YAML):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: dev
spec:
  containers:
    - name: nginx
      image: nginx:latest
```

Pods in different namespaces can communicate directly via Pod IPs since the ip's are for cluster network which is flat . However, when accessing a Service across namespaces using curl, the fully qualified domain name (FQDN) (e.g., svc-test.default.svc.cluster.local) must be used. This is based on the DNS search domains defined in /etc/resolv.conf.

---

## 🧱 Multi Container Pod (init container, app container, sidecar/helper container) 

In a multi-container Pod, all containers share the same network namespace, storage volumes, and resource allocations. This means they can communicate over localhost and access the same data volumes. Typically, 

- an init container runs setup tasks
- the main app container handles the core workload
- sidecar/helper containers provide supporting functionality (like logging or syncing).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  # Shared volume between containers
  volumes:
    - name: shared-data
      emptyDir: {}

  # Init container runs once before app and sidecar start
  initContainers:
    - name: init-setup
      image: busybox
      command: ['sh', '-c', 'echo "Init done" > /data/init.log']
      volumeMounts:
        - name: shared-data
          mountPath: /data

  containers:
    # Main application container
    - name: app
      image: nginx
      ports:
        - containerPort: 80
      volumeMounts:
        - name: shared-data
          mountPath: /usr/share/nginx/html
      env:
        - name: ENVIRONMENT
          value: "dev"         # Injects environment variable into the container

    # Sidecar container that tails the init log file
    - name: sidecar
      image: busybox
      command: ["sh"]
      args: ["-c", "tail -f /data/init.log"]   # Runs tail command as container's main process
      volumeMounts:
        - name: shared-data
          mountPath: /data
```

---

## 🛰️ Daemonset

A Deployment creates a specified number of Pod replicas already mentioned and distributes them across available nodes based on scheduling policies and resource availability by schedular. However, the replica count is fixed unless manually updated or managed by an HPA (Horizontal Pod Autoscaler).     

A DaemonSet, on the other hand, ensures that exactly one Pod runs on each node (or on selected nodes using labels or taints/tolerations). If a new node is added, the DaemonSet automatically schedules a Pod on it. Likewise, if a node is deleted, the Pod scheduled by the DaemonSet on that node is also automatically removed.        

Example Pods managed by DaemonSets: kube-proxy, kubelet, datadog-agent, node-exporter, filebeat etc.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemonset
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

By default, Pods created by a DaemonSet are scheduled on all worker nodes, but not on control plane nodes. This is because control plane nodes are typically tainted with:  

node-role.kubernetes.io/control-plane:NoSchedule       
      
This taint prevents regular (custom) workloads from being scheduled on them. Since DaemonSet Pods are considered regular workloads, they do not get scheduled on control plane nodes unless the Pod is configured with a matching toleration to explicitly allow it.

---

## ⏰ CronJob

A CronJob in Kubernetes allows you to run Jobs on a repeating schedule, similar to how Linux cron works. It's ideal for tasks like: Backups, Log rotation, Scheduled reporting, Cleanup scripts.
         
- First  *   -  Minute         (0-59)             
- Second *   -  Hour           (0-23)             
- Third  *   -  Day of Month   (1-31)            
- Fourth *   -  Month          (1-12)        
- Fifth  *   -  Day of Week    (0-7)  → Sunday = 0 or 7        

📄 Sample CronJob YAML:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cron
spec:
  schedule: "*/10 * * * *"           # Runs every 10 minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: hello
              image: busybox
              args:
                - /bin/sh
                - -c
                - echo "Hello from CronJob at $(date)"
          restartPolicy: OnFailure
```

Job - A Job in Kubernetes is used to run a task to completion — meaning it runs a pod (or multiple pods) until the task is successfully finished, and then it doesn't restart it again (unless configured to retry on failure). It's ideal for tasks like: Batch processing, One-time scripts, Database cleanup or patching, Sending notification emails etc.     

A Kubernetes Job runs a pod to completion (e.g., batch task), and when it finishes successfully, the pod shows Completed 0/1 — meaning it’s done and not running. 

📄 Sample YAML for a Job:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: example-job
spec:
  template:
    spec:
      containers:
        - name: example
          image: busybox
          command: ["sh", "-c", "echo Hello Kubernetes! && sleep 10"]
      restartPolicy: OnFailure
```

---

## 📌 Kubernetes Scheduler

The Kubernetes Scheduler is a control plane component responsible for assigning Pods to Nodes. When you create a Pod (directly or via a Deployment, Job, etc.), the Pod initially has no Node assigned. The scheduler selects the best Node based on available resources, constraints, and rules. 

The Kubernetes Scheduler is itself a Pod, but it's not scheduled like normal Pods it is a Static pod. A Static Pod is managed directly by the kubelet, not by the Kubernetes API Server.     

It's defined via a YAML file on disk (usually under /etc/kubernetes/manifests/) which consists of 
- etcd.yaml
- kube-scheduler.yaml
- kube-controller-manager.yaml
- kube-apiserver.yaml

The kubelet automatically monitor and starts it and if the manifest is missing then the componnents and the functionality it provides is also not there in the cluster. The kube-scheduler, like other control plane components (e.g., kube-apiserver, kube-controller-manager), is a static pod.       

By default, Kubernetes automatically schedules Pods using the kube-scheduler. But you can manually schedule a Pod by explicitly assigning it to a specific Node using the nodeName field.

📄 Sample YAML for Manual Scheduling:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: manual-pod
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: worker-node-1  # 👈 Directly assign the Pod to this node
```

It will get created even if scheduler is not there. The kube-scheduler skips Pods that already have a nodeName set.

---

## 🏷️ Labels

Key-value pairs attached to Kubernetes objects (like Pods, Deployments, etc.) and the purpose for this is Used to organize, group, and select objects.

Sample yaml for Labels:
```yaml
metadata:
  labels:
    app: nginx
    tier: frontend
```

---

## 🎯 Selectors

Mechanism to filter/select objects based on labels. Used in Services, ReplicaSets, Deployments, Network Policies etc.

matchLabels - A type of label selector that matches exactly on the key-value pair.    

```yaml
selector:
  matchLabels:
    app: nginx
```

nodeSelector - nodeSelector is used in a Pod spec to pin or schedule a Pod to specific nodes that have matching labels.

```bash
kubectl label nodes node1 gpu=false
```

Sample yaml with nodeSelector:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-gpu-pod
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    gpu: "false"
```

---

## 💡 Taint

A taint is applied to a node to repel certain Pods from being scheduled onto it, unless those Pods tolerate the taint.

🧾 Taint on Node:
```bash
kubectl taint nodes node1 gpu=true:NoSchedule
```

This taint means: Do not schedule any Pod on node1 unless it has a matching toleration.

---

## 🛡️ Toleration

A toleration is applied to a Pod to allow it to be scheduled onto a node with a matching taint.      

Effects - 

- NoSchedule - Pod will not be scheduled unless it tolerates the taint     
- PreferNoSchedule - Tries to avoid scheduling, but not strictly enforced      
- NoExecute - Evicts running Pods that don't tolerate the taint       

🧾 Example: Toleration in a Pod
```yaml
tolerations:
- key: "gpu"
  operator: "Equal"
  value: "true"
  effect: "NoSchedule"
```

If a Pod doesn't have a toleration for a taint on a node, and no other nodes are available without that taint, then the Pod will go into the Pending state.

🧹 Remove a Taint from a Node
```bash
kubectl taint nodes node1 gpu=true:NoSchedule-
```

---

## ⚔️ Taint & Toleration vs nodeSelector

Taints and tolerations prevent pods from landing on certain nodes unless the pod explicitly tolerates the taint. However, toleration alone does not guarantee scheduling on that node — the scheduler still prefers untainted nodes first.

In contrast, nodeSelector is a hard rule: it forces the pod to schedule only on nodes with matching labels.

To guarantee scheduling on a tainted node, use both:

- toleration → allows the tainted node
- nodeSelector → targets it directly

---

## 🧲 Node Affinity

When you need more complex or multiple matching rules (like OR/AND logic, preferred rules, multiple labels, etc.), that's where Node Affinity comes in.

Node Affinity is a mechanism in Kubernetes that allows you to constrain which nodes your pod is eligible to be scheduled on, based on labels assigned to nodes. It's similar to nodeSelector, but more expressive and flexible.

There are two main types of node affinity:

1) RequiredDuringSchedulingIgnoredDuringExecution: This is a hard rule. If the condition is not met, the pod will not be scheduled. Example: disk=ssd means only nodes with label disk=ssd are allowed.

2) PreferredDuringSchedulingIgnoredDuringExecution: This is a soft rule. Scheduler will try to honor the condition, but if not possible, it can still schedule the pod elsewhere.

Node Affinity Rules Are Only Evaluated at Scheduling Time - Any changes made to node labels during the execution of a pod do not affect already running pods.

Sample YAML for Node Affinity

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-example
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disk
                operator: In
                values:
                  - ssd
  containers:
    - name: nginx
      image: nginx
```

Use Taints & Tolerations together with Node Affinity for fine-grained pod placement control.

- Taints say: "Don’t come here unless you’re allowed."
- Node Affinity says: "Please try to go there if possible."

---

## 💾 Resource Requests & Limits

1) Request - The minimum amount of CPU/Memory the pod needs to be scheduled on the node. Think: "Reservation".        

2) Limit - The maximum amount of CPU/Memory the pod is allowed to use of the node. Think: "Ceiling".

Example YAML for Resource Requests & Limits:
```ymal
resources:
  requests:
    cpu: "200m"         # 0.2 vCPU
    memory: "256Mi"     # 256 MiB RAM
  limits:
    cpu: "500m"         # Max 0.5 vCPU
    memory: "512Mi"     # Max 512 MiB RAM
```

⚠️ What Happens on Insufficient Resources?    

- ❌ During Scheduling - If the requested CPU/Memory can't be fulfilled by any node, then we say Nodepool is hit and the pod stays in Pending. Error: 0/3 nodes are available: insufficient memory.
  
- 🧠 During Runtime: Exceeds Memory Limit - If the container tries to use more than its memory limit → OOMKilled. Kubernetes terminates the container immediately. Event: OOMKilled (Out Of Memory error) for example: You set limit to 512Mi but app uses 600Mi → Boom! 💥 & Monitor OOMKilled events with:

```bash
kubectl get events --field-selector reason=OOMKilling
```

- 🔄 Exceeds CPU Limit - Container is throttled, not killed. CPU usage is capped, leading to slower performance. This happens silently unless you monitor it. Always define requests and limits to: Help the scheduler place your pods wisely and Prevent a single pod from hogging all node resources.

---

## 📊 Stress Testing - Enable Resource Monitoring (Metrics Server)

Kubernetes doesn’t expose resource usage (CPU/Memory) by default. You need to deploy the Metrics Server to collect and access these stats.

```yaml
# metrics-server-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    k8s-app: metrics-server
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
        - name: metrics-server
          image: registry.k8s.io/metrics-server/metrics-server:v0.6.4
          args:
            - --kubelet-insecure-tls
            - --kubelet-preferred-address-types=InternalIP
          ports:
            - containerPort: 4443
```

```bash
kubectl apply -f metrics-server-deployment.yaml
```

Verify Metrics:
```bash
kubectl top node     # Shows node-level resource usage
kubectl top pod      # Shows pod-level resource usage
```

So now we can use the polinux/stress image to simulate CPU and memory pressure for testing Kubernetes behavior under stress — useful for observing OOMKilled, throttling, or autoscaling reactions.    

Sample YAML for Stress Testing using polinux/stress:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: stress-test
spec:
  containers:
    - name: stress
      image: polinux/stress
      command: ["stress"]
      args: ["--cpu", "2", "--vm", "1", "--vm-bytes", "512M", "--timeout", "60s"]
      resources:
        requests:
          memory: "256Mi"
          cpu: "500m"
        limits:
          memory: "300Mi"
          cpu: "1"
```

This will help you visualize stress handling, especially if you combine it with: kubectl top node (to see pressure at node level) and Event logs for OOMKilled:
```bash
kubectl apply -f stress-test.yaml
kubectl top pod stress-test
kubectl describe pod stress-test
kubectl logs stress-test
```
     
In Kubernetes, when a Pod consumes more resources than its defined limits, the system chooses to kill or throttle the Pod rather than impacting the entire node.

---

## 📈 Kubernetes Autoscaling

Kubernetes autoscaling is the process of automatically adjusting the number of Pods or resources in a cluster based on real-time workload demands. These are main types:

1) Horizontal Pod Autoscaler (HPA) (scale out / scale in) - Scales the number of Pod replicas up or down based on CPU/memory usage or custom metrics. Example: If CPU > 70%, more Pods are added to balance the load.

2) Vertical Pod Autoscaler (VPA) (scale up / scale down) - Adjusts the resource requests/limits (CPU/memory) of existing Pods. It recommends or automatically updates values based on observed usage over time.

3) Cluster Autoscaler (scale out / scale in) - Scales the underlying nodes in the cluster (like EC2, GKE nodes) by adding/removing nodes when Pods can't be scheduled due to insufficient resources.

4)  Event & Schedule-Based Autoscaling - KEDA (CNCF project) enables event-driven autoscaling in Kubernetes using external sources like Kafka, RabbitMQ, Prometheus, etc. It also supports cron-based autoscaling, allowing workloads to scale up/down at scheduled times — ideal for predictable traffic spikes.

In Kubernetes, you can define multiple objects in a single YAML file by separating them with ---.

Together, these autoscaling components help ensure performance, cost-efficiency, and optimal resource utilization in dynamic workloads. Kubernetes autoscaling applies to both workloads (Pods) and infrastructure (Nodes) to ensure performance and efficiency under changing demand.     

HPA (Horizontal Pod Autoscaler) is a native feature in Kubernetes and only requires the Metrics Server to function. VPA and Cluster Autoscaler are official projects but not shipped by default so they must be installed and configured manually.

---

## ⬆️⬇️ Horizontal Pod Autoscaler (HPA)

The HPA controller checks CPU metrics every 15 seconds (by default) using metrics from the Metrics Server. If the average CPU usage across pods exceeds the threshold, it scales up, and scales down if it's below the threshold.

Imperative Way (CLI):
```bash
kubectl autoscale deployment nginx-deployment \
  --cpu-percent=50 \
  --min=2 \
  --max=5
```

📄 Declarative Way (YAML) :
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 5
  targetCPUUtilizationPercentage: 50
```

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: memory-based-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

Generate load on a Kubernetes service (for example, likely a deployment called php-apache) to test Horizontal Pod Autoscaler (HPA) functionality. Perfect for stress testing and observing how HPA reacts to CPU load.

```bash
kubectl run -i --tty load-generator \
  --rm \
  --image=busybox:1.28 \
  --restart=Never \
  -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
```

Watch Pods Live:
```bash
kubectl get pods -w
kubectl get pods -n default -w
```

---

## 🔍 Health Probes 

In Kubernetes, health probes are used to monitor the health and availability of containers running inside Pods. These probes help the kubelet determine whether a container is Running correctly, Ready to accept traffic, Needs to be restarted.

Types of Health Probes - 

1) Liveness Probe - Checks if the container is alive, restart the container if it fails.
2) Readiness Probe - Checks if the container is ready to serve traffic, if it fails then Container is removed from the Service endpoints.
3) Startup Probe - Checks if the container has started successfully and helps avoid killing slow-starting containers (especially legacy apps).

⚙️ Probe Configuration Methods - Health checks (probes) can be performed using three types of methods to determine the state of a container:

1) HTTP GET request - Sends an HTTP GET request to a specific path and port of the container.
2) TCP Socket check - Opens a TCP connection to a specified port of the container.
3) Exec command inside the container - Executes a command inside the container. If the command exits with status 0, it’s healthy; anything else is unhealthy.

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5

startupProbe:
  httpGet:
    path: /start
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

 TCP and Exec probes:
 ```yaml
readinessProbe:
      tcpSocket:
        port: 80           # Check if TCP port is open
      initialDelaySeconds: 5
      periodSeconds: 10

livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy      # Run a command inside the container
      initialDelaySeconds: 5
      periodSeconds: 10
```

---

## 🔑 ConfigMaps

When using the same environment variable across multiple Pods, hardcoding in each Pod spec becomes inefficient. Instead, use ConfigMaps to centralize and manage environment variables.

```yaml
env:
 - name: ENVIRONMENT
   value: "dev"         # Injects environment variable into the container
```

Imperative Way:
```bash
kubectl create configmap my-config \
  --from-literal=FIRSTNAME=John \
  --from-literal=LASTNAME=Doe
```

Sample YAML to create ConfigMap:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  FIRSTNAME: John
  LASTNAME: Doe
```

```bash
kubectl apply -f configmap.yaml
```

 Inject ConfigMap into Pod (envFrom):
 ```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "env && sleep 3600"]
    envFrom:
      - configMapRef:
          name: my-config
```

This will expose all key-value pairs from my-config as environment variables inside the container.

---

## 💬📄 SSL/TLS

HTTP (Without SSL/TLS - Insecure)

1) Client → Server: Sends a GET /login request in plain text.    
2) Server → Client: Responds with login page.    
3) Client → Server: Submits username/password in plain text.    
4) Hacker (in middle): Intercepts traffic and reads credentials (sniffed).    

🔐 Symmetric Encryption (Single Key) - Same key is used to encrypt and decrypt data. Both parties must share the same secret key securely. Problem: Key distribution is risky (if a hacker gets the key, they can decrypt everything).

🔐 Asymmetric Encryption (Public / Private Key) - 

Uses a key pair:

1) Public Key – can be shared with anyone.
2) Private Key – must be kept secret.

🔑 How It Works: Encrypt with Public Key → Decrypt with Private Key. Commonly used in SSH, HTTPS (TLS), etc.

Example with SSH:

1) You run ssh-keygen to generate:
2) id_rsa → Private Key (keep it secret)
3) id_rsa.pub → Public Key (share with server)
4) Server adds your public key to ~/.ssh/authorized_keys

When you connect: Server challenges you. Your private key proves your identity (without sending password).     

✅ Encryption Example:
- A client wants to send secure data to a server.
- The server shares its public key.
- The client uses the public key to encrypt data.
- The server uses its private key to decrypt the data.        

This ensures:
- Only the server (holder of the private key) can read the data.
- Even if intercepted, the encrypted data is useless without the private key.       

🧑‍💻 Man-in-the-Middle Attack Risk - Even with public/private keys, a hacker could trick you by:

Acting as the server:

1) Sending their own public key instead of the real one
2) Then they decrypt your data and forward it — you won’t know!

📜 Solution: **Certificates** (TLS/SSL) - A certificate is a digitally signed proof of identity issued by a trusted Certificate Authority (CA).

Prevents fake servers from impersonating real ones. Your browser (or client) trusts known CAs. If a server sends a self-signed or fake certificate, the client gets a warning. This stops the attacker from impersonating the server (🔐 secure HTTPS).

🔁 Flow:

1) Client connects to server over HTTPS.
2) Server sends its certificate.
3) Client checks that certificate is:
4) Signed by a valid CA
5) Matches the domain if not expired
6) If valid → Secure TLS session starts 🔐
7) If invalid → ⚠️ Warning (Untrusted Connection)

---

## 🔏 TLS certificates in Kubernetes

Kubernetes secures internal communication using TLS certificates to ensure confidentiality and trust between components.

1) Kubectl ↔️ API Server: Communication is encrypted. kubectl uses a client certificate or token stored in ~/.kube/config.
2) API Server ↔️ Kubelet (Node): TLS verifies and encrypts the connection when accessing logs, exec, or managing pods on nodes.
3) Core Components: Control plane components like kube-scheduler, kube-controller-manager, and kube-proxy use certificates stored in /etc/kubernetes/pki.

You can manually generate TLS certificates using openssl or manage them via tools like cert-manager.

```bash
# View client cert used by kubectl
kubectl config view --raw
```

Example TLS secret creation:

```bash
kubectl create secret tls apiserver-tls \
  --cert=apiserver.crt --key=apiserver.key -n kube-system
```

Use TLS secrets to mount certificates into pods for secure communication. Kubernetes ensures zero-trust by default, enabling strong authentication and encryption across the cluster.

---

## 🔑 Authentication & Authorization in Kubernetes

By default, Kubernetes uses the ~/.kube/config file to authenticate and authorize users like kubectl.

🧾 **Authentication** - This verifies who you are.

When you run a kubectl command, it uses the credentials in your ~/.kube/config file. These credentials can be:

1) A client certificate
2) A bearer token
3) A username/password
4) Or a cloud provider plugin

Example (inside ~/.kube/config):

```yaml
users:
- name: dev-user
  user:
    client-certificate: ~/.kube/dev-user.crt
    client-key: ~/.kube/dev-user.key
```

🔐 **Authorization** - This checks what you're allowed to do. Once authenticated, Kubernetes uses an authorization module (like Role-Based Access Control) to determine if the user can perform the requested action. RBAC is a security mechanism in Kubernetes that controls who can do what on cluster resources.

Kubernetes supports multiple authorization modes—like RBAC, ABAC, Webhook, and Node—to control access to resources, with RBAC being the most widely used and recommended.

🧾 RBAC Components:

1) Role – Defines a set of permissions (verbs like get, create, delete) on resources (pods, services, etc.) within a namespace.
2) RoleBinding – Grants a Role to a user/service account in a namespace.
3) ClusterRole – Like Role, but applies cluster-wide.
4) ClusterRoleBinding – Grants a ClusterRole to users/subjects across all namespaces.

Role:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""] # indicates the core API group
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

RoleBinding: We have to bind the above role to the user or user groups and it is namespace scoped.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: dev
subjects:
- kind: User
  name: dev-user        # Must match a user from your kubeconfig
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

ClusterRole – Grants access to resources across the whole cluster (not limited to a single namespace).

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

ClusterRoleBinding – Binds the above ClusterRole to a user/service account across all namespaces.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-all-pods
subjects:
- kind: User
  name: cluster-user           # Must match a user from kubeconfig
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Test Access:

```bash
kubectl auth whoami
kubectl auth can-i list pods --as dev-user --namespace dev
```

## 🤖 ServiceAccount 

In Kubernetes, users can be of various types such as human users (developers, admins) or non-human users like applications, bots, or CI/CD tools. For non-human users, we commonly use ServiceAccounts. These are Kubernetes identities intended for applications running inside pods.    

Each ServiceAccount automatically receives a secret token, which is mounted into the pod and used to authenticate with the API server. This token can also be used to authenticate with private container registries (like ACR, ECR, Docker Hub) by creating a Kubernetes Secret and referencing it in a pod or deployment.        

🛠️ Sample: Create a ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-bot
  namespace: default
```

🔑 Create Docker Registry Secret (for private ACR)

```yaml
kubectl create secret docker-registry acr-secret \
  --docker-server=<acr-login-server> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email> \
  --namespace=default
```

🔗 Link Secret to ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-bot
  namespace: default
imagePullSecrets:
- name: acr-secret
```

🚀 Use ServiceAccount in a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  serviceAccountName: app-bot
  containers:
  - name: app
    image: <acr-login-server>/my-app:latest
```

✅ Direct Secret Usage in Pod (imagePullSecrets)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: <acr-login-server>/my-app:latest
  imagePullSecrets:
  - name: acr-secret
```

---

## 🌐 Kubernetes Networking Overview

Kubernetes networking ensures that all Pods, Services, and external clients can communicate reliably. It follows flat networking (vnet), meaning:       

When a Pod is Created:
1) The CNI plugin is called (e.g., Calico, Azure CNI).
2) The CNI assigns an IP, connects the Pod to the virtual network.
3) CoreDNS helps the Pod resolve other service names. 

Pods and Flat Networking:
1) Every pod gets a unique internal IP.
2) All pods can talk to each other without NAT.
3) Networking is flat like a virtual LAN.

🔌 **CNI (Container Network Interface)** - CNI is how Kubernetes handles Pod networking. When a Pod is created, the Kubelet calls a CNI plugin to set up network interfaces, assign IPs, and connect the Pod to the network. The CNI used depends on the Kubernetes provider—for example,the default one is Coredns and for Azure it uses Azure CNI, which allows full Pod-to-Pod communication by default. Custom CNIs like Calico or Cilium can enforce network policies to restrict traffic.

🛠 What CNI Handles:

1) Pod-to-Pod communication across nodes
2) Assigns IP addresses (IPAM)
3) Creates veth (virtual ethernet) pairs

**Kubernetes Services (L4)** - Kubernetes Services expose Pods:

1) ClusterIP – internal only
2) NodePort – exposed on each node's IP + port
3) LoadBalancer – exposes via external LB (cloud-based)

🌉 **Ingress (L7 - HTTP/HTTPS routing)** - Ingress manages application-layer (L7) traffic routing in Kubernetes, primarily for HTTP and HTTPS. It enables you to expose multiple backend services through a single external IP using routing rules based on hostnames and paths (e.g., /api, /app). Ingress requires an Ingress Controller (e.g., NGINX, Traefik, HAProxy) to function.         
🔁 Ingress Traffic Flow:
- External client sends an HTTP/HTTPS request to the cluster's Ingress Controller.
- The Ingress Controller inspects the host and path in the request.
- Based on defined Ingress rules, it forwards the request to the appropriate backend Service.
- TLS termination (decrypting HTTPS) and certificate management (e.g., via cert-manager) are typically handled at the Ingress level.

Sample YAML for ingress:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
    - host: mysite.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80
```

🔁 Basic Network Terminology:
- North-South traffic: Traffic that enters or exits your mesh (e.g., from internet to your cluster or vice versa).
- East-West traffic: Internal communication between services within the mesh (e.g., microservice A calling microservice B).

**🛡 Istio: Advanced Routing and Security**

Key Istio Components:
1) Ingress Gateway: Handles incoming (north-south) traffic.
2) Egress Gateway: Controls outbound internet access.
3) VirtualService: Handles traffic routing inside mesh.
4) DestinationRule: Define traffic policies (e.g., load balancing).

---

## 🧩 Where Gateway and VirtualService Fit

🛣️ **Gateway (Kubernetes + Istio object)** : Defines how external traffic enters the mesh via the Ingress Gateway (usually HTTP, HTTPS, TCP). These two Istio resources work together to define how traffic enters and moves within the mesh.

It configures the Istio Ingress Gateway, defining ports, protocols, TLS, etc.

Sample YAML for Gateway:
```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: my-ingress-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
    - port:
        number: 80
        name: http
        protocol: HTTP
      hosts:
        - "myapp.example.com"
```

🚏 **VirtualService** : Defines how traffic is routed once inside the mesh. 
- Works with Gateway for external traffic, or standalone for internal service-to-service (east-west) traffic.
- Supports routing rules, retries, fault injection, traffic splitting (canary), etc.

Sample YAML for VirtualService:
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp-route
spec:
  hosts:
    - "myapp.example.com"
  gateways:
    - my-ingress-gateway
  http:
    - route:
        - destination:
            host: myapp
            port:
              number: 80
```

---

## ☸️ Kubeadm

Identify Your Deployment Purpose:
- Proof of Concept (PoC) / Local Development: Use tools such as Minikube or Kind.
- Managed Kubernetes on Cloud Providers: Options include AKS (Azure Kubernetes Service), EKS (Elastic Kubernetes Service), or GKE (Google Kubernetes Engine).
- Self-Managed Kubernetes: Deploy on platforms like VirtualBox, cloud-based virtual machines, or on-premises infrastructure using bare metal or VMware.


Kubeadm is a tool provided by Kubernetes to bootstrap a production-ready cluster quickly and easily. It automates the setup of critical components like:

1) kube-apiserver
2) kube-controller-manager
3) kube-scheduler
4) etcd (the key-value store)
5) TLS certificate generation and configuration

 What kubeadm Does:
 
1) Initializes the control plane (kubeadm init)
2) Joins worker nodes to the cluster (kubeadm join)
3) Sets up default cluster networking

You use kubeadm when you want to manually build a Kubernetes cluster (e.g., on VMs, bare metal, or cloud instances) without a fully managed kubernetes services like GKE, EKS, or AKS.

---

## 🛠️☸️ Kubernetes Cluster Setup with kubeadm (3-Node Cloud Setup)
 
🖥️ Infrastructure: 3 VMs (Cloud)
- 1 Master (Control Plane)
- 2 Worker Nodes

✅ Prerequisites (All Nodes):
- Disable swap
- Update kernel params (iptables, IP forwarding)
- Install: Container runtime (containerd)
- Install: runc
- Install: CNI plugins
- Install: kubeadm, kubelet, kubectl

🚀 On Master Node:
- kubeadm init with --pod-network-cidr
- Setup kubeconfig (.kube/config)
- Deploy CNI (e.g., Calico)

🔗 On Worker Nodes:
-Use kubeadm join command from master to join the cluster

#### 🔐 Kubernetes Network Ports and Component Flow :

📌 Control Plane (Master Node):
| Protocol | Direction | Port Range  | Purpose                    | Used By                     |
|----------|-----------|-------------|-----------------------------|----------------------------|
| TCP      | Inbound   | 6443        | Kubernetes API server       | All                        |
| TCP      | Inbound   | 2379–2380   | etcd server client API      | kube-apiserver, etcd       |
| TCP      | Inbound   | 10250       | Kubelet API                 | Self, Control plane        |
| TCP      | Inbound   | 10259       | kube-scheduler              | Self                       |
| TCP      | Inbound   | 10257       | kube-controller-manager     | Self                       |

📌 Worker node(s)
| Protocol | Direction | Port Range     | Purpose              | Used By               |
|----------|-----------|----------------|----------------------|-----------------------|
| TCP      | Inbound   | 10250          | Kubelet API          | Self, Control plane   |
| TCP      | Inbound   | 10256          | kube-proxy           | Self, Load balancers  |
| TCP      | Inbound   | 30000–32767    | NodePort Services    | All                   |
| UDP      | Inbound   | 30000–32767    | NodePort Services    | All                   |


#### ⚙️ Multi-Node Kubernetes Cluster Setup Using Kubeadm (1 CP + 2 Workers)

**Pre-Setup (All Nodes):**
  
- Disable Swap
```bash
swapoff -a && sudo sed -i '/ swap / s/^/#/' /etc/fsta
```

- Enable Required Kernel Modules and Sysctl
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
EOF

sudo sysctl --system
```

- Install Container Runtime (containerd):
```bash
curl -LO <containerd-tar.gz> && sudo tar Cxzvf /usr/local <tar.gz>
curl -LO <containerd.service> && sudo mv containerd.service /usr/local/lib/systemd/system/

sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

sudo systemctl daemon-reload
sudo systemctl enable --now containerd
```

- Install Runc and CNI Plugins
```bash
# Runc
curl -LO <runc.amd64>
sudo install -m 755 runc.amd64 /usr/local/sbin/runc

# CNI Plugins
curl -LO <cni-plugins.tgz>
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin <cni-plugins.tgz>
```

- Install Kubernetes Binaries
```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gpg

curl -fsSL <key> | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt install -y kubelet=1.29.6-1.1 kubeadm=1.29.6-1.1 kubectl=1.29.6-1.1 --allow-downgrades
sudo apt-mark hold kubelet kubeadm kubectl

kubeadm version
kubelet --version
kubectl version --client
```

- Configure crictl:
```bash
sudo crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock
```

**🧠 Control Plane Node Only:**

-  Initialize Control Plane by kubeadm:
```bash
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=<CP-IP> --node-name master
```

- Set Up Kubeconfig:
```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

- Install Calico Network Plugin
```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml
kubectl apply -f custom-resources.yaml
```

**⚙️ Worker Nodes (Both)**:

Repeat: Disable swap, load modules, install containerd, runc, CNI, kubelet, kubeadm, kubectl

- Join the Cluster: Run the kubeadm join command provided at the end of kubeadm init. It will look like this:
```bash
sudo kubeadm join <CP-IP>:6443 \
  --token <your-token> \
  --discovery-token-ca-cert-hash sha256:<your-hash>
```

- You can re-generate it on the control plane with:
```bash
kubeadm token create --print-join-command
```

**✅ Post-Setup Validation**

- All nodes should show Ready status and pods should be running.
```bash
kubectl get nodes
kubectl get pods -A
```

**📄 Copy Kubeconfig from Master**

To allow kubectl on the worker, copy the admin kubeconfig:
```bash
# From the master node:
scp master-user@<CP-IP>:/etc/kubernetes/admin.conf ~/admin.conf

# On the worker node:
mkdir -p $HOME/.kube
mv ~/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

Now you can run kubectl commands on the worker using the copied kubeconfig.

---

## 🐳 Docker Persistent Storage

📦 Why You Need Persistent Storage:
- Ephemeral Containers: By default, a container’s writable layer is lost whenever the container is removed or crashes. Anything you mkdir, write, or modify inside / vanishes with the container.
- Shared Access: Often you want multiple containers (or pods) to read/write the same data, and survive restarts or re-scheduling.
- Volumes to the Rescue: Docker volumes (and Kubernetes PersistentVolumes) mount a host directory or an external storage backend into the container’s filesystem. The container’s writable layer remains read-only except at the mount point, so your data lives on beyond container lifetimes.

⚙️ Under the Hood: 
- The storage driver handles writing data inside the container's writable layer, which is ephemeral. To make data persistent across container restarts, a volume driver is used to store data outside the container's lifecycle.
- Storage Drivers: Docker uses a storage driver to manage the container’s layered filesystem:
- overlay2 (default on most Linux distros)
- zfs, btrfs, vfs (less common, advanced use cases)

☁️ Cloud-specific drivers integrate external block/object storage:
- AWS: EBS, EFS
- GCP: Persistent Disk
- Azure: Azure Disk, Blob (via CSI)

**Docker Volume**

Create a volume
```bash
docker volume create data_vol
```

Inspect Available Volumes
```bash
docker volume ls
```

Locate Data on the Host: By default, local volumes live under /var/lib/docker/volumes/
```bash
ls /var/lib/docker/volumes/data_vol/_data
```

Run a Container with the Volume Mounted
```bash
docker run -d \
  --name webapp \
  -v data_vol:/usr/share/nginx/html \
  -p 3000:80 \
  nginx
```

Verify Persistence:
```bash
# Create a test file inside the running container
docker exec -it webapp bash -c "echo 'Hello, world!' > /usr/share/nginx/html/index.html"

# Restart or remove & recreate the container
docker rm -f webapp
docker run -d \
  --name webapp \
  -v data_vol:/usr/share/nginx/html \
  -p 3000:80 \
  nginx

# The index.html you created is still there:
curl http://localhost:3000
# => Hello, world!
```

Even if the container is destroyed, your data in data_vol remains intact and re-mounted on any new container using the same volume.

**Bind Mounts**

Unlike named volumes (managed by Docker and stored under /var/lib/docker/volumes), bind mounts map a specific directory or file from the host machine directly into the container.

🧠 Key Differences:
- Bind Mounts use absolute paths on the host.
- They provide more control and visibility into the underlying data.
- Changes in the host directory immediately reflect in the container and vice versa.
- Useful during development or when you need to share config/data/logs with the host.

So -v can be used for bind mounts as well as named volumes. But for clarity and best practice, especially with bind mounts, it’s recommended to use the --mount flag — which is more explicit and readable.

```bash
# Using shorthand -v (bind mount)
docker run -v /home/ubuntu/mydata:/usr/share/nginx/html -d -p 8080:80 nginx
```

```bash
# Using --mount (recommended for clarity)
docker run --mount type=bind,source=/home/ubuntu/mydata,target=/usr/share/nginx/html -d -p 8080:80 nginx
```

--- 

## 📁 Kubernetes Volumes

To preserve data or share data between containers in the Pod, Kubernetes Volumes are used. 
- Volume Mounts tells the container where to mount the volume inside its filesystem.
- Volumes defines the source of the storage.

📄 Sample YAML with emptyDir{} - This is a type of volume that is created empty when the Pod is assigned to a Node. It lives as long as the Pod lives. Data is lost if the Pod is deleted (but not if the container inside restarts).
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-pod
spec:
  containers:
    - name: redis
      image: redis
      volumeMounts:
        - name: redis-storage
          mountPath: /data/redis  # This is where data will be written inside the container
  volumes:
    - name: redis-storage
      emptyDir: {}  # Volume that lives as long as the Pod does
```

**Persistent Volume (PV)**
- A cluster resource that represents actual physical storage (e.g., disk, NFS, cloud storage).
- Created and managed by Storage Admin.
- Can be pre-provisioned or dynamically provisioned via a StorageClass.

**Persistent Volume Claim (PVC)**
- A request for storage made by a Pod.
- Created by Application or K8s Admin.
- Specifies size, access mode, and optionally a StorageClass.

**StorageClass**
- Defines how volumes are provisioned dynamically.
- Includes details like provisioner (e.g., AWS EBS, GCE PD), reclaim policy, volume type, etc.

📘 Real-World Example Flow:
- Storage admin defines a StorageClass and may pre-create a 100Gi PV (or rely on dynamic provisioning).
- A developer creates a PVC requesting 10Gi.
- The PVC is bound to the PV (if available).
- The remaining 90Gi of the PV is not usable unless using volume expansion or dynamic provisioning.
- If another PVC requests 150Gi or mismatched access mode, it remains in Pending.

PV Access Modes:
- ReadWriteOnce (RWO) - Mounted as read-write by a single node
- ReadOnlyMany (ROX) - Mounted as read-only by many nodes
- ReadWriteMany (RWX) - Mounted as read-write by many nodes
- ReadWriteOncePod (RWOP) - Mounted read-write by a single Pod (K8s 1.22+)

Define a StorageClass:
```yaml
# storage-class.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/no-provisioner  # or ebs.csi.aws.com, etc.
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain  # Options: Retain, Delete, Recycle
```

- 🔁 Retain means volume must be manually deleted.
- 🗑️ Delete auto deletes PV when PVC is deleted.
- 🔁 Recycle(Deprecated) Tries to clean and reuse PV.

Create Persistent Volume (Manually)
```yaml
# pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-100gi
spec:
  capacity:
    storage: 100Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast-storage
  hostPath:
    path: "/mnt/data"
```

Create Persistent Volume Claim
```yaml
# pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redis-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-storage
  resources:
    requests:
      storage: 10Gi
```

Use PVC in a Pod
```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-pod
spec:
  containers:
    - name: redis
      image: redis
      volumeMounts:
        - name: redis-vol
          mountPath: /data/redis
  volumes:
    - name: redis-vol
      persistentVolumeClaim:
        claimName: redis-pvc
```

🔎 Behavior: Binding & Allocation:
- Once PVC (10Gi) is created, it binds to PV (100Gi).
- Now, 90Gi is unused but cannot be reused unless volume supports volume expansion or dynamic slicing.
- If another PVC requests 150Gi, or a different access mode, it will remain in Pending.

📋 Check State
```yaml
kubectl get pv
kubectl get pvc
kubectl describe pvc redis-pvc
```

🧠 Summary
- Storage Admin: creates PVs or configures StorageClass.
- K8s Admin/Dev: creates PVCs in workloads.
- Kubernetes binds PVC ↔ PV if size & mode match.
- Reclaim policy defines behavior after PVC is deleted.

---

## 🌐 DNS (Domain Name System)

It’s like the phonebook of the internet. DNS helps translate human-readable names into IP addresses — just like saving names for phone numbers in your phone. You type a website like:
```bash
www.youtube.com
```

But computers talk using IP addresses, like:
```bash
142.250.64.78
```

- Domain - A domain is a unique name on the internet. Example: google.com → domain
- Subdomain - A subdomain is a smaller part of a domain. Example: www.google.com

**🧭 DNS Resolution (Step-by-Step)**

When you type www.google.com, here's what happens:
1) Your browser checks if it already knows the IP.
2) If not, it asks your DNS resolver (like Cloudflare or Google DNS).
3) That resolver asks the root server (one of the 13).
4) The root replies: “Ask the .com server.”
5) .com server replies: “Ask google.com's nameserver.”
6) Google’s nameserver replies with the IP address.
7) Your browser connects to that IP.
8) 💡 All this happens in milliseconds!

There are 13 root DNS servers in the world (named A to M). These are the starting point for all DNS lookups. They know where to find Top-Level Domains (TLDs) like .com, .org, .net, etc. They don’t know IP addresses of websites directly, but they know where to look next.

**🗂️ DNS Records Explained**

1) A Record - Maps a domain to an IP address.
2) CNAME (Canonical Name) - Maps a name to another name (not IP).
3) NS (Name Server) - Tells which name servers are responsible for the domain.
4) MX (Mail Exchange) - Defines the mail servers for a domain.

**☁️ Cloudflare DNS and Google DNS**

These are public DNS resolvers — free services that anyone can use.

1) ✅ Google DNS - IPs: 8.8.8.8 and Fast and globally available
2) ✅ Cloudflare DNS - IP: 1.1.1.1 and Focuses on speed and privacy

You can set these DNS servers in your device or router to speed up and secure your browsing.
 
---

## 🧭 DNS in Kubernetes

In a Kubernetes cluster, DNS is critical for service discovery—pods communicate with each other using service names (like my-service) instead of IPs. This is made possible by CoreDNS, the default DNS service running as a pod in the kube-system namespace.      

Without CoreDNS running, pods cannot resolve service names, though they can still reach other pods by IP. Each pod also has a /etc/hosts file that allows it to resolve its own hostname, but not others.      

Previously, Kubernetes used kube-dns, but it’s now deprecated in favor of CoreDNS, which is faster, more modular, and easier to configure. When a service is created, it automatically gets a DNS name like my-service.default.svc.cluster.local, and pods can access it using just my-service if they're in the same namespace.

Check if CoreDNS is running:
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

Check your pod's DNS setup:
```bash
kubectl exec -it <pod-name> -- cat /etc/resolv.conf
kubectl exec -it <pod-name> -- cat /etc/hosts
```

Test DNS resolution from a pod:
```bash
kubectl exec -it <pod-name> -- nslookup my-service
kubectl exec -it <pod-name> -- dig my-service.default.svc.cluster.local
```

---

## 🚀 How to Upgrade a Multi-Node Kubernetes Cluster (Step-by-Step)

**🔄 Why Upgrade?**
- New Kubernetes releases bring features, bug fixes, and security patches.
- Only the 3 most recent minor versions are officially supported.
- For example, if the current version is v1.31.x, then only v1.31.x, v1.30.x, and v1.29.x are supported. Anything older is end-of-life (no patches) will gone come. It will still work but not "recommendable".


**🔢 Semantic Versioning in Kubernetes**
- Kubernetes follows the format: MAJOR.MINOR.PATCH
- You can only upgrade one minor version at a time.
- Allowed: v1.28.x → v1.29.x
- Not Allowed: v1.28.x → v1.30.x

**🛠 Upgrade Flow Overview**
- Primary Control Plane Node (first to upgrade)
- Other Control Plane Nodes 
- Worker Nodes (one by one)   

💡 A LoadBalancer or virtual IP is used to balance requests across control plane nodes. That’s why you can safely upgrade them one at a time.

**⚙️ Commands for Worker Node Upgrade**

1) Drain the Node:
```bash
kubectl drain worker1 --ignore-daemonsets --delete-emptydir-data
```
- drain: Evicts pods from the node.
- cordon: Automatically marks node as unschedulable.
- If pods are part of a Deployment, they get re-created on another node.
- If pods are standalone (no controller), they are deleted.

2) Upgrade Kubelet and Kubeadm on Node:
```bash
# Upgrade kubeadm to match new control plane version
sudo apt-get install -y kubeadm=1.29.3-00

# Verify plan
sudo kubeadm upgrade node

# Upgrade kubelet
sudo apt-get install -y kubelet=1.29.3-00

# Restart kubelet
sudo systemctl restart kubelet
```

3) Uncordon the Node:
```bash
kubectl uncordon worker1
```

- Node is now schedulable again.
- Pods can be placed back on it.

**🔁 Upgrade Strategies**
| Strategy        | Description                                                                |
|-----------------|----------------------------------------------------------------------------|
| All at Once     | Upgrade all nodes together (risky for production).                         |
| Rolling Update  | One node at a time, maintains high availability (recommended).             |
| Blue-Green      | Bring up new upgraded nodes, test, and switch traffic, then delete old.    |

---

## 💾 ETCD Backup And Restore

- etcd is the central database for Kubernetes.
- It stores all cluster state: nodes, namespaces, pods, ConfigMaps, Secrets, service accounts, etc.
- It’s a key-value store used by the Kubernetes control plane (specifically the API server).

Misconception: Does kubectl get all -A -o yaml > backup.yaml back up everything?      
- ❌ No, it only backs up resource configurations (like Deployments, Services, etc.) — not the full cluster state.
- It does not back up: Cluster metadata, Secrets securely, Internal Kubernetes objects, Pod data (e.g., files inside volumes).

**Real Backup: Use ETCD Snapshot**

If you're managing your own Kubernetes cluster (e.g., via kubeadm), you have direct access to ETCD and can take full snapshots:
```bash
# Backup ETCD (Run on Control Plane node)
# This command saves a complete snapshot of the cluster’s state to /backup/etcd-snapshot.db.
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

To restore:
```bash
ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db \
  --data-dir /var/lib/etcd-from-backup
```

🔁 You’ll need to reconfigure the kube-apiserver to point to this restored directory if you're doing a full cluster restore.

**☁️ What if You're Using Managed Kubernetes (like EKS, AKS, GKE)?**   
- You do not have direct access to ETCD.
- Instead, use: Velero – a tool to back up and restore Kubernetes resources and volumes. Cloud provider’s native backup tools (e.g., AWS Backup for EKS).
```bash
# Velero CLI example
velero backup create my-cluster-backup --include-namespaces default
```

📦 What About Pod Data?
- ETCD does not store application data inside pods (like uploaded files, DB data).
- That data is in Volumes (e.g., emptyDir, hostPath, PersistentVolume).
- You must back up Persistent Volumes separately (using Velero or volume snapshots).

## 👀 Kubernetes Logging and Monitoring

By default, Kubernetes does not provide logging and monitoring. To expose metrics like CPU and memory utilization, we enable the Metrics Server, which is a CNCF project.
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

We get the logs of the pod. From there, you can redirect these logs to Splunk, Fluent Bit, or any third-party tool for centralized logging.
```bash
kubectl logs <pod-name>
```

So when the kubectl command stops working, it usually means that the API server is not listening on its port. In such cases, we troubleshoot the problem at the container level because we know that the API server itself runs as a static pod. By default, Kubernetes uses containerd as the container runtime (not Docker). So instead of using docker ps, we use crictl ps to inspect the running containers. The issue might be due to image pulling failures or problems starting the API server container.
```bash
# Check running containers when API server is not responding
crictl ps
```

## 🔬 Advanced Kubectl Commands

So when you run kubectl get nodes, kubectl sends a GET request to the API server. The API server responds with a JSON payload, which kubectl then converts into a human-readable format. When we need additional details, such as specific properties, we can look directly at the raw JSON payload.
```bash
kubectl run nginx --image=nginx:latest --dry-run=client -o json > pod.json
kubectl run nginx --image=nginx:latest --dry-run=client -o json > pod.yaml
```






