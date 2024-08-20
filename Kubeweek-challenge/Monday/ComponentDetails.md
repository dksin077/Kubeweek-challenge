## Kubernetes  Components Details


### Master Node Components::

#### MasterNode or Control Plane:- 
- The master node is the control plane of Kubernetes. It is responsible for managing the state of the cluster, scheduling and deploying applications, and scaling them as needed. The master node consists of several components.

#### APIServer:-
- The API server is the central management point for the Kubernetes cluster. It handles API requests from users and other components in the system. It validates and processes these requests to the backend components.

#### ETCD:-
- ETCD is a distributed key-value store that stores the configuration data for the entire cluster. It is used to store the state of the system, including information about running pods, services, and replication controllers.

#### Scheduler:-
- The scheduler is responsible for assigning pods to nodes based on resource availability and other constraints.

#### ControllerManager:-
- The controller manager is responsible for managing the various controllers in the system, which are responsible for ensuring that the desired state of the system is maintained. It monitor the cluster to make sure everything is running smoothly. If something goes wrong (e.g., a Pod crashes), they work to fix it.
  
#### Cloud Controller Manager:-
- This is a specialized component that allows Kubernetes to interact with the underlying cloud provider, like AWS or Azure. It helps in tasks like setting up load balancers and persistent storage.


## Worker Node Components::
### Worker Nodes:
- These nodes are the worker machines that run containerized applications. Each node runs one or more containers, and the node is responsible for managing the containers on that machine. Each node runs several components, including.

#### Kubelet:-
- The kubelet is an agent that runs on each node and is responsible for managing the containers on that node. It communicates with the API server to receive instructions about which containers to run and how to run them.

#### Kube-Proxy:-
- The kube-proxy is responsible for managing the network connectivity between pods and services. Kube-Proxy manages the network rules on each node to make sure traffic is properly routed to the correct pods.
- kube-proxy focused on routing the services incoming/outgoing traffic to specific pod and providing the load balancing.
  
## Other Components::
#### Kubectl:-
- A command-line tool for dealing with Kubernetes clusters is called kubectl. On a Kubernetes cluster, it is used to deploy, monitor, and manage applications. On most operating systems, including Linux, macOS, and Windows, kubectl can be installed because it is a component of the official Kubernetes distribution.

### Some Other Components:-
#### Pod:-
- A pod is the smallest deployable unit in Kubernetes. It is a logical host for one or more containers, and each pod runs on a single node. Pods can contain one or more containers that share the same network namespace and file system.

#### Service:-
- The main purpose of service is to provide a stable IP address or DNS name, that's makes communication very easy between the pods, even as Pods are created, destroyed, or replaced.
- Types of Servies
  - ClusterIP:- Expose the Pods with in a cluster.
  - NodePort:- Expose the Pods Outside the cluster, When no reuirement of load balancing.
  - LoadBalancer:- Expose the Pods outside the cluster with loadbalancer for this we used cloud providers like AWS,GCP and Azure.
  - ExternalName:- used to expose external service which is outside the cluster like RDS it can be resolved by Kubernetes pods directly.
- Function: Service discover the pods through the labels and expose them with the consistent IP or DNS.

#### Volume:- 
- This is like an external hard-drive that can be attached to a Pod to store data.

#### Namespace:-
- A way to divide cluster resources among multiple users or teams. Think of it as having different folders on a shared computer, where each team can only see their own folder.
Imagine you’re managing a large Kubernetes cluster with several applications. You want to keep everyone’s belongings organized and ensure that the resources are used efficiently. So, you assign different namespaces to different applications.

#### Ingress:-
- Think of this as the "front door" for external access to your applications, controlling how HTTP and HTTPS traffic should be routed to your services.
Conatiner Runtime:-
- This is the software used to run containers. Docker is commonly used, but other runtimes like containerd can also be used.

#### CNI(Container Network Interface):-
- CNI work is to manage pod-level networking, So that every pod get there IP address and easily communicate with each other.
  
#### CRI-O(Conatiner Runtime Interface Object):-
- CRI-O is a tool that helps to manage containers in Kubernetes. It helps to run docker,conatinerd etc in containers.

#### Short note for the communication between master and worker node:
- Control Plane (Management Office): This is where the factory managers (API Server) are. They send instructions to the workers (Kubelets) on the factory floor (worker nodes).
- Worker Nodes (Factory Floor): This is where the actual work happens. The workers (Kubelets) report back to the management office (API Server) about their progress and any issues.
- CNI (Communication System): This is like the internal phone system that allows everyone to communicate effectively within the factory and beyond.
- Kube-Proxy (Routing System): This ensures that messages (network traffic) get to the right worker (Pod) on the factory floor.
