## Install Kubernetes cluster on Ubuntu server 22.04

### Prerequistic
- Number of Nodes - 2
- CPU Core - 2
- RAM - 3gb
- Disk - 20gb

### Install for All Nodes

```
sudo apt update
sudo apt upgrade -y
sudo reboot
```

### Set hostname and update hosts file
- ON Master
```
sudo hostnamectl set-hostname "k8s-master"

sudo vi /etc/hosts
192.168.100.3 k8s-master
192.168.100.4 k8s-worker
```

- On worker

```
sudo hostnamectl set-hostname "k8s-worker"

sudo vi /etc/hosts
192.168.100.3 k8s-master
192.168.100.4 k8s-worker
```

### Disable swap
```
sudo swapoff -a

#search swap and comment
sudo vi /etc/fstab
```

### Load the following kernel modules on all the nodes:
```
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

### Set the following Kernel parameters for Kubernetes.

```
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Then reloadsysctl
sudo sysctl --system
```

### Run the following commands on all nodes to install containerd service

```
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y containerd.io

# Once installed, we add the containerd configuration.

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### Install Kubernetesi, Run the following commands on all nodes

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### Initialize cluster using kubeadm, Just run the command below on the master node

```
# Replace your cidr, In which 10.10.0.0/16is the CIDR of the pod network, you can change it as needed.

sudo kubeadm init \
  --pod-network-cidr=10.10.0.0/16 \
  --control-plane-endpoint=k8s-master
```

### If the run is successful, the result will be as follows

```
master@k8s-master:~$ sudo kubeadm init \
  --pod-network-cidr=10.10.0.0/16 \
  --control-plane-endpoint=k8s-master
[init] Using Kubernetes version: v1.26.1
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.100.3]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master localhost] and IPs [192.168.100.3 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master localhost] and IPs [192.168.100.3 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 7.503422 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node k8s-master as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node k8s-master as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
[bootstrap-token] Using token: daii9y.g4dq24u6irkz4pt0
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap,RBAC Roles
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regularuser:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listedat:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following asroot:

  kubeadm join k8s-master:6443 --token daii9y.g4dq24u6irkz4pt0 \
        --discovery-token-ca-cert-hash sha256:58b9cc96ed57a5797fddea653756dbda830efbff55b720a10cffb3948d489148 \
        --control-plane

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s-master:6443 --token daii9y.g4dq24u6irkz4pt0 \
        --discovery-token-ca-cert-hash sha256:58b9cc96ed57a5797fddea653756dbda830efbff55b720a10cffb3948d489148
```

### Next, execute the commands below on the master node

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Let's run the command to check the cluster status.

```
kubectl cluster-info
kubectl get nodes
```

### Return result:

```
master@k8s-master:~$ kubectl cluster-info

Kubernetes control plane is running at https://k8s-master:6443
CoreDNS is running at https://k8s-master:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

master@k8s-master:~$ kubectl get nodes

NAME                         STATUS     ROLES           AGE   VERSION
k8s-master   NotReady   control-plane   55m   v1.26.1
```

### NOTE:-We see that the control plane is running and currently there is only the master node, we will proceed to add worker nodes to this cluster.

#### Note: The commands below run only on worker nodes.

#### Then you can join any number of worker nodes by running the following on each as root:

```
kubeadm join k8s-master:6443 --token daii9y.g4dq24u6irkz4pt0 \
        --discovery-token-ca-cert-hash sha256:58b9cc96ed57a5797fddea653756dbda830efbff55b720a10cffb3948d489148

master@k8s-master:~$ kubectl get nodes
NAME          STATUS     ROLES           AGE   VERSION
k8s-master    NotReady   control-plane   58m   v1.26.1
k8s-worker    NotReady   <none>          87s   v1.26.1

```

### Install Calico, Pod Network for Kubernetes cluster

```
curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml -O

# Open the downloaded file and find the sectionCALICO_IPV4POOL_CIDR

- name: CALICO_IPV4POOL_CIDR
  value: "10.10.0.0/16"

# Disable file logging so `kubectl logs` works.

- name: CALICO_DISABLE_FILE_LOGGING
  value: 'true'

```

### Then we install Calico on Kubernetes cluster:

```
kubectl apply -f calico.yaml

# We will check if calico has been deployed successfully, make sure coredns also running.

master@k8s-master:~$ kubectl get pods -n kube-system

NAME                                                 READY   STATUS    RESTARTS   AGE
calico-kube-controllers-57b57c56f-ptddp              1/1     Running   0          2m44s
calico-node-5fqml                                    1/1     Running   0          2m44s
calico-node-llfjq                                    1/1     Running   0          2m44s
calico-node-vw78h                                    1/1     Running   0          2m44s
coredns-787d4945fb-n7s9t                             1/1     Running   0          62m
coredns-787d4945fb-rs9mj

# If the status is Running, it means the deployment was successful.

master@k8s-master:~$ kubectl get nodes
NAME          STATUS   ROLES           AGE     VERSION
k8s-master    Ready    control-plane   62m     v1.26.1
k8s-worker    Ready    <none>          5m53s   v1.26.1

```
