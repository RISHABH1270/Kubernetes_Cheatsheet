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

 ## 🛡️ Namespaces

Namespaces provide an additional layer of isolation to help organize and separate Kubernetes objects. If no namespace is specified, the object is created in the default namespace. Kubernetes system components and internal objects are created in the kube-system namespace.      

Resources within the same namespace can communicate using short names or direct Pod IPs. For cross-namespace access, objects must be referenced using their fully qualified domain names (FQDN), as defined by the cluster’s internal DNS settings.

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

Imperative Way (Direct CLI Command)
```bash
kubectl create namespace dev
```

Declarative Way (Using YAML Manifest)
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

```bash
kubectl apply -f namespace.yaml
```

To delete a namespace in Kubernetes, ⚠️ This will delete all resources within the dev namespace, so use with caution.
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

## 🧱 Multi Container Pod (init container, app container, sidecar/helper container) 

In a multi-container Pod, all containers share the same network namespace, storage volumes, and resource allocations. This means they can communicate over localhost and access the same data volumes. Typically, an init container runs setup tasks, the main app container handles the core workload, and sidecar/helper containers provide supporting functionality (like logging or syncing).


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

## 🛰️ Daemonset

A Deployment creates a specified number of Pod replicas already mentioned and distributes them across available nodes based on scheduling policies and resource availability. However, the replica count is fixed unless manually updated or managed by an HPA (Horizontal Pod Autoscaler).     

A DaemonSet, on the other hand, ensures that exactly one Pod runs on each node (or on selected nodes using labels or taints/tolerations). If a new node is added, the DaemonSet automatically schedules a Pod on it. Likewise, if a node is deleted, the Pod scheduled by the DaemonSet on that node is also automatically removed.        

Example Pods managed by DaemonSets: kube-proxy, datadog-agent, node-exporter, filebeat etc.

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

By default, Pods created by a DaemonSet are scheduled on all worker nodes, but not on control plane nodes. This is because control plane nodes are typically tainted with: node-role.kubernetes.io/control-plane:NoSchedule       
      
This taint prevents regular (custom) workloads from being scheduled on them. Since DaemonSet Pods are considered regular workloads, they do not get scheduled on control plane nodes unless the Pod is configured with a matching toleration to explicitly allow it.

## ⏰ CronJob

A CronJob in Kubernetes allows you to run Jobs on a repeating schedule, similar to how Linux cron works. It's ideal for tasks like: Backups, Log rotation, Scheduled reporting, Cleanup scripts.
         
First  *   -  Minute         (0-59)             
Second *   -  Hour           (0-23)             
Third  *   -  Day of Month   (1-31)            
Fourth *   -  Month          (1-12)        
Fifth  *   -  Day of Week    (0-7)  → Sunday = 0 or 7        

📄 Sample CronJob YAML

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

📄 Sample YAML for a Job

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

## 📌 Kubernetes Scheduler

The Kubernetes Scheduler is a control plane component responsible for assigning Pods to Nodes. When you create a Pod (directly or via a Deployment, Job, etc.), the Pod initially has no Node assigned. The scheduler selects the best Node based on available resources, constraints, and rules. 

The Kubernetes Scheduler is itself a Pod, but it's not scheduled like normal Pods it is a Static pod. A Static Pod is managed directly by the kubelet, not by the Kubernetes API Server. It's defined via a YAML file on disk (usually under /etc/kubernetes/manifests/) which consists of etcd.yaml, kube-scheduler.yaml, kube-controller-manager.yaml, kube-apiserver.yaml etc and the kubelet automatically monitor and starts it and if the manifest is missing then the componnents and the functionality it provides is also not there in the cluster. The kube-scheduler, like other control plane components (e.g., kube-apiserver, kube-controller-manager), is a static pod.

By default, Kubernetes automatically schedules Pods using the kube-scheduler. But you can manually schedule a Pod by explicitly assigning it to a specific Node using the nodeName field.

📄 Sample YAML: Manual Scheduling

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

## 🏷️ Labels

Key-value pairs attached to Kubernetes objects (like Pods, Deployments, etc.) and the purpose for this is Used to organize, group, and select objects.

Sample yaml for Labels

```yaml
metadata:
  labels:
    app: nginx
    tier: frontend
```

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

Sample yaml with nodeSelector
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

## 💡 Taint

A taint is applied to a node to repel certain Pods from being scheduled onto it, unless those Pods tolerate the taint.

🧾 Example: Taint on Node

```bash
kubectl taint nodes node1 gpu=true:NoSchedule
```

This taint means: Do not schedule any Pod on node1 unless it has a matching toleration.

## 🛡️ Toleration

A toleration is applied to a Pod to allow it to be scheduled onto a node with a matching taint.      

Effects - 

NoSchedule - Pod will not be scheduled unless it tolerates the taint     
PreferNoSchedule - Tries to avoid scheduling, but not strictly enforced      
NoExecute - Evicts running Pods that don't tolerate the taint       

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

## Taint & Toleration vs nodeSelector

Taints and tolerations prevent pods from landing on certain nodes unless the pod explicitly tolerates the taint. However, toleration alone does not guarantee scheduling on that node — the scheduler still prefers untainted nodes first.

In contrast, nodeSelector is a hard rule: it forces the pod to schedule only on nodes with matching labels.

To guarantee scheduling on a tainted node, use both:

toleration → allows the tainted node      

nodeSelector → targets it directly

## Node Affinity

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

1) Taints say: "Don’t come here unless you’re allowed."     

2) Node Affinity says: "Please try to go there if possible."

## Resource Requests & Limits

Request - The minimum amount of CPU/Memory the pod needs to be scheduled on the node. Think: "Reservation".     

Limit - The maximum amount of CPU/Memory the pod is allowed to use of the node. Think: "Ceiling".

Example YAML for Resource Requests & Limits 

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

1)  ❌ During Scheduling - If the requested CPU/Memory can't be fulfilled by any node, then we say Nodepool is hit and the pod stays in Pending. Error: 0/3 nodes are available: insufficient memory.

2) 🧠 During Runtime: Exceeds Memory Limit - If the container tries to use more than its memory limit → OOMKilled. Kubernetes terminates the container immediately. Event: OOMKilled (Out Of Memory error) for example: You set limit to 512Mi but app uses 600Mi → Boom! 💥 & Monitor OOMKilled events with:

```bash
kubectl get events --field-selector reason=OOMKilling
```

3) 🔄 Exceeds CPU Limit - Container is throttled, not killed. CPU usage is capped, leading to slower performance. This happens silently unless you monitor it. Always define requests and limits to: Help the scheduler place your pods wisely and Prevent a single pod from hogging all node resources.


## Stress Testing - 📊 Enable Resource Monitoring (Metrics Server)

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

Verify Metrics - 

```bash
kubectl top node     # Shows node-level resource usage
kubectl top pod      # Shows pod-level resource usage
```

So now we can use the polinux/stress image to simulate CPU and memory pressure for testing Kubernetes behavior under stress — useful for observing OOMKilled, throttling, or autoscaling reactions.    

Sample YAML for Stress Testing using polinux/stress

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

This will help you visualize stress handling, especially if you combine it with: kubectl top node (to see pressure at node level) and Event logs for OOMKilled.    

```bash
kubectl apply -f stress-test.yaml
kubectl top pod stress-test
kubectl describe pod stress-test
kubectl logs stress-test
```
     
In Kubernetes, when a Pod consumes more resources than its defined limits, the system chooses to kill or throttle the Pod rather than impacting the entire node.

## ⚙️ Kubernetes Autoscaling

Kubernetes autoscaling is the process of automatically adjusting the number of Pods or resources in a cluster based on real-time workload demands. These are main types:

1) Horizontal Pod Autoscaler (HPA) (scale out / scale in) - Scales the number of Pod replicas up or down based on CPU/memory usage or custom metrics. Example: If CPU > 70%, more Pods are added to balance the load.

2) Vertical Pod Autoscaler (VPA) (scale up / scale down) - Adjusts the resource requests/limits (CPU/memory) of existing Pods. It recommends or automatically updates values based on observed usage over time.

3) Cluster Autoscaler (scale out / scale in) - Scales the underlying nodes in the cluster (like EC2, GKE nodes) by adding/removing nodes when Pods can't be scheduled due to insufficient resources.

4)  Event & Schedule-Based Autoscaling - KEDA (CNCF project) enables event-driven autoscaling in Kubernetes using external sources like Kafka, RabbitMQ, Prometheus, etc. It also supports cron-based autoscaling, allowing workloads to scale up/down at scheduled times — ideal for predictable traffic spikes.

In Kubernetes, you can define multiple objects in a single YAML file by separating them with ---.

Together, these autoscaling components help ensure performance, cost-efficiency, and optimal resource utilization in dynamic workloads. Kubernetes autoscaling applies to both workloads (Pods) and infrastructure (Nodes) to ensure performance and efficiency under changing demand.     

HPA (Horizontal Pod Autoscaler) is a native feature in Kubernetes and only requires the Metrics Server to function. VPA and Cluster Autoscaler are official projects but not shipped by default — they must be installed and configured manually.

## Horizontal Pod Autoscaler (HPA)

The HPA controller checks CPU metrics every 15 seconds (by default) using metrics from the Metrics Server. If the average CPU usage across pods exceeds the threshold, it scales up, and scales down if it's below the threshold.

Imperative Way (CLI)

```bash
kubectl autoscale deployment nginx-deployment \
  --cpu-percent=50 \
  --min=2 \
  --max=5
```

📄 Declarative Way (YAML) 

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

Watch Pods Live

```bash
kubectl get pods -w
kubectl get pods -n default -w
```

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

 TCP and Exec probes - 

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

## ConfigMaps

When using the same environment variable across multiple Pods, hardcoding in each Pod spec becomes inefficient. Instead, use ConfigMaps to centralize and manage environment variables.

```yaml
env:
 - name: ENVIRONMENT
   value: "dev"         # Injects environment variable into the container
```

Imperative Way

```bash
kubectl create configmap my-config \
  --from-literal=FIRSTNAME=John \
  --from-literal=LASTNAME=Doe
```

Sample YAML to create ConfigMap

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

## SSL/TLS

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

🧑‍💻 Man-in-the-Middle Attack Risk - Even with public/private keys, a hacker could trick you by:

Acting as the server:

1) Sending their own public key instead of the real one
2) Then they decrypt your data and forward it — you won’t know!

📜 Solution: Certificates (TLS/SSL) - A certificate is a digitally signed proof of identity issued by a trusted Certificate Authority (CA).

Prevents fake servers from impersonating real ones. Your browser (or client) trusts known CAs. If a server sends a self-signed or fake certificate, the client gets a warning. This stops the attacker from impersonating the server (🔐 secure HTTPS).

🔁 Flow:

1) Client connects to server over HTTPS.
2) Server sends its certificate.
3) Client checks that certificate is:
4) Signed by a valid CA
5) Matches the domain if not expired
6) If valid → Secure TLS session starts 🔐
7) If invalid → ⚠️ Warning (Untrusted Connection)

## TLS certificates in Kubernetes

Kubernetes secures internal communication using TLS certificates to ensure confidentiality and trust between components.

1) kubectl ↔️ API Server: Communication is encrypted. kubectl uses a client certificate or token stored in ~/.kube/config.
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

## 🔑 Authentication & Authorization in Kubernetes

By default, Kubernetes uses the ~/.kube/config file to authenticate and authorize users like kubectl.

🧾 Authentication - This verifies who you are.

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

🔐 Authorization - This checks what you're allowed to do. Once authenticated, Kubernetes uses an authorization module (like Role-Based Access Control) to determine if the user can perform the requested action. RBAC is a security mechanism in Kubernetes that controls who can do what on cluster resources.

Kubernetes supports multiple authorization modes—like RBAC, ABAC, Webhook, and Node—to control access to resources, with RBAC being the most widely used and recommended.

🧾 RBAC Components:

1) Role – Defines a set of permissions (verbs like get, create, delete) on resources (pods, services, etc.) within a namespace.
2) ClusterRole – Like Role, but applies cluster-wide.
3) RoleBinding – Grants a Role to a user/service account in a namespace.
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

RoleBinding: We have to bind the above role to the user.

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

Test Access:

```bash
kubectl auth can-i list pods --as dev-user --namespace dev
```



