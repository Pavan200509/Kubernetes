# Kubernetes Complete Learning & Practical Guide 

This repository contains my end-to-end Kubernetes learning journey, covering Kubernetes architecture, control plane components, worker node components, networking, storage, scheduling, deployments, scaling, monitoring, troubleshooting,
and real-world application deployment.

# Kubernetes (K8s) 
It is an open-source container orchestration platform that automates the deployment, scaling, networking, and management of containerized applications across multiple servers. The two core parts in kubernetes are
Master(controle plane) and Worker Nodes.

## Complete Flow of a request from User to POD.

kubectl
  ↓
API Server
  ↓
etcd
  ↓
Controller Manager
  ↓
ReplicaSet
  ↓
Pod Object
  ↓
Scheduler
  ↓
API Server + etcd (stores node assignment)
  ↓
Kubelet
  ↓
Container Runtime
  ↓
Pod Running
  ↓
Service
  ↓
Kube-proxy
  ↓
Traffic reaches Pods
  ↓
Ingress (optional for external access)

# Operations of each and every Component in above flow.

### kubectl

The process starts when the user executes a command using kubectl, such as applying a YAML file or running a command directly. kubectl does not communicate with etcd or any Kubernetes component directly. It sends the request to the
API Server. The request contains the desired state of the application, such as the number of Pods, container image, ports, and other configurations.

### API Server

The API Server receives the request from kubectl and validates it. It checks whether the request is valid and whether the user has permission to perform the action. If everything is valid, the API Server stores the desired state in
etcd. The API Server acts as the central communication hub of Kubernetes. Every major component such as the Controller Manager, Scheduler, Kubelet, and Kube-proxy communicates with the cluster through the API Server.

### etcd

etcd acts as the database of Kubernetes. It stores all cluster information, including Deployments, ReplicaSets, Pods, Services, Nodes, and scheduling decisions. etcd does not communicate with other components directly. Components 
retrieve and update data through the API Server, which reads from and writes to etcd.

### Controller Manager

Once the desired state is stored in etcd, the Controller Manager continuously watches the API Server for cluster information. The API Server provides the desired state stored in etcd and the current state reported by the worker nodes.
The Controller Manager compares both states. If the desired state says that three Pods should exist but currently no Pods exist, the Controller Manager takes action. For a Deployment, it creates a ReplicaSet object and stores it through
the API Server into etcd.

### ReplicaSet

The ReplicaSet object contains information about how many Pods should exist at all times. The ReplicaSet itself does not create containers or run Pods on nodes. It simply represents the requirement that a specific number of Pods must
be maintained. If the desired number is three Pods and only two exist, the Controller Manager notices this through the ReplicaSet object and creates another Pod object through the API Server.

### Scheduler

Once Pod objects are created, they do not have any worker node assigned yet. The Scheduler continuously watches the API Server for unscheduled Pods. To make a scheduling decision, it requests node information from the API Server. 
The API Server provides node details that were previously reported by Kubelets and stored in etcd, including available CPU, memory, labels, taints, and other scheduling information. Based on these details, the Scheduler selects the
most suitable node. It then updates the Pod object with the selected node assignment through the API Server. This scheduling decision is stored in etcd by the API Server.

### Kubelet

Each worker node runs a Kubelet. The Kubelet continuously watches the API Server for Pods assigned to its node. It does not communicate directly with the Scheduler or etcd. Once the Scheduler assigns a Pod to a node and that information
is stored through the API Server, the Kubelet notices that a Pod has been assigned to its node. The Kubelet retrieves the Pod specification from the API Server and starts the Pod creation process.

### Container Runtime

The Kubelet does not run containers itself. Instead, it sends instructions to the Container Runtime, such as containerd or CRI-O. The Container Runtime receives the Pod specification from the Kubelet, pulls the required container
image from a registry if necessary, creates the container environment, and starts the application process. Once the containers start successfully, the Pod becomes operational.
o
### Worker Node Status Reporting

After the Pod starts, the Kubelet continuously monitors the Pod and the worker node. It collects information such as Pod health, CPU usage, memory usage, node status, and container status. The Kubelet sends this information to the API
Server, which stores it in etcd. This information is later used by the Scheduler and Controller Manager when making decisions.

### Service

Once the application Pods are running, users should not access Pod IPs directly because Pod IPs can change when Pods are recreated. A Service object is created separately and assigned a stable virtual IP called a ClusterIP by Kubernetes.
The Service continuously identifies matching Pods using labels. If a Pod is deleted and recreated with a new IP address, the Service automatically updates its list of available Pods. Users and applications communicate with the Service IP
rather than individual Pod IPs.

### Kube-proxy

Kube-proxy continuously watches the API Server for information about Services and the Pods associated with them. The API Server provides Service and Pod information that is stored in etcd. Based on this information, Kube-proxy creates 
networking rules using iptables or IPVS on the worker node. When traffic arrives at a Service IP, Kube-proxy uses these rules to forward the request to one of the healthy Pods behind the Service. If a Pod is removed or recreated,
Kube-proxy automatically updates its routing rules.

### Ingress

If the application needs to be accessed from outside the Kubernetes cluster, an Ingress resource is created. The Ingress watches incoming HTTP or HTTPS requests and forwards them to the appropriate Service. The Service then forwards the
request to one of the Pods through Kube-proxy. This allows users to access applications using domain names and secure HTTPS connections.

**Only the API Server communicates directly with etcd. Every other Kubernetes component gets or updates information through the API Server.** This is the key concept that ties the entire Kubernetes flow together.
