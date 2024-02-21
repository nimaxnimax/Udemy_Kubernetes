
![alt text](cover.jpg)

Udemy Course >> Kubernetes Helm Server Deployment - VPS/Cloud DevOps Part 1

Kubernetes Helm Server Infrastructure Deployment - Cluster Installation and Configuration on VPS/Cloud - DevOps Part 1


**********

Tips/Tricks/Notes/Commands URL Link: https://github.com/nimaxnimax/Udemy_Kubernetes

Instructor & Courses >> https://www.udemy.com/user/adrian-fischer-infotech/


**********

How to Install Kubernetes Cluster on Ubuntu 20 and 22

Kubernetes has become the de facto container orchestration platform, empowering developers and system administrators to manage and scale containerized applications effortlessly. If you’re an Ubuntu 20 or 22 user eager to harness the power of Kubernetes, you’ve come to the right place.

A Kubernetes cluster consists of worker nodes on which application workload is deployed and a set up master nodes which are used to manage worker nodes and pods in the cluster.


Prerequisites

We are using one master node and one worker node. You can configure 3 to 5 workers in your lab or on your VPS/Cloud infa. Following are system requirements on each node
- Minimal install Ubuntu 20 or 22
- Minimum 2GB RAM or more
- Minimum 2 CPU core / or 2 vCPU
- 20 GB free disk space
- Sudo user with admin rights
- Internet connectivity on each node


Lab Setup
- Master Node: Public/Private IP Address
- First Worker Node: Public/Private IP Address


**********

Ubuntu 22 Installation and Config >>>

Set hostname on Each Node

Login to to master node and set hostname using hostnamectl command

```bash
sudo hostnamectl set-hostname "master"
exec bash
```

On the worker nodes run >> If you are configuring more than one worker change the hostname for each of them

```bash
sudo hostnamectl set-hostname "w1"
exec bash
```

Add the following entries in /etc/hosts file on each node (Replace IP Addresses With Your Public/Private IPs)

```bash
sudo vi /etc/hosts
```

```bash
MASTER_IP   master
WORKER1_IP  w1
```

Real Environment >> Disable swap & Add kernel Parameters

Execute beneath swapoff and sed command to disable swap. Make sure to run the following commands on all the nodes.

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

Load the following kernel modules on all the nodes

```bash
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```

```bash
tail /etc/modules-load.d/containerd.conf
sudo modprobe overlay
sudo modprobe br_netfilter
```

Set the following Kernel parameters for Kubernetes run beneath tee command

```bash
sudo tee /etc/sysctl.d/kubernetes.conf <<EOT
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOT
```

Reload the above changes run

```bash
sudo sysctl --system
```

Install Containerd Runtime

In this guide we are using containerd runtime for our Kubernetes cluster. So, to install containerd first install its dependencies.

```bash
sudo apt update -y
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```

Enable docker repository

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

Now run following apt command to install containerd

```bash
sudo apt update
sudo apt install -y containerd.io
```

Configure containerd so that it starts using systemd as cgroup.

```bash
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

Restart and enable containerd service

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

Add Apt Repository for Kubernetes

Kubernetes package is not available in the default Ubuntu 22.04 package repositories. So we need to add Kubernetes repository. run following command to download public signing key

Ubuntu 22 >>

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Ubuntu 20 >>

```bash
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

Next, run following echo command to add Kubernetes apt repository.

Ubuntu 22 >>

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Ubuntu 20 >>

```bash
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

Install Kubectl, Kubeadm and Kubelet

Post adding the repositories, install Kubernetes components like kubectl, kubelet and Kubeadm utility on all the nodes. Execute following set of commands

```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Install Kubernetes Cluster on Ubuntu 22.04

Now, we are all set to initialize Kubernetes cluster. Run the following Kubeadm command on the master node only.

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

If any error >> bypass >>

```bash
sudo kubeadm init --ignore-preflight-errors=NumCPU,Mem,Swap --pod-network-cidr=10.244.0.0/16
```

If any problem... Reset and start again.

```bash
sudo kubeadm reset
```

After the initialization is complete, you will see a message with instructions on how to join worker nodes to the cluster. Make a note of the kubeadm join command for future reference.

So, to start interacting with cluster, run following commands on the master node

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

next, try to run following kubectl commands to view cluster and node status

```bash
kubectl cluster-info
kubectl get nodes
```

If any problem to check errors >>>

```bash
kubectl cluster-info dump
```

Join Worker Nodes to the Cluster

On each worker node, use the kubeadm join command you noted down earlier after initializing the master node. It should look something like this:

Example >> 

```bash
sudo kubeadm join MASTER_IP --token TOKEN \
   --discovery-token-ca-cert-hash sha256:HASH
```

Check the nodes status from master node using kubectl command

```bash
kubectl get nodes
```

As we can see nodes status is ‘NotReady’, so to make it active. We must install CNI (Container Network Interface) or network add-on plugins like Calico, Flannel and Weave-net.

Install Calico Network Plugin

A network plugin is required to enable communication between pods in the cluster. Run following kubectl command to install Calico network plugin from the master node

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml
```

Verify the status of pods in kube-system namespace

```bash
kubectl get pods -n kube-system
```

Perfect, check the nodes status as well

```bash
kubectl get nodes
```

Great, above confirms that nodes are active node. Now, we can say that our Kubernetes cluster is functional.

In your lab or not production server deployment >> if not enough memory >> after installation enable swap or config it >> Swap Config

```bash
sudo fallocate -l 4G /swapfile
ls -anp /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**********

Ubuntu 20 Installation and Config >>>

Set hostname on Each Node

Login to to master node and set hostname using hostnamectl command

```bash
sudo hostnamectl set-hostname "master"
exec bash
```

On the worker nodes run >> If you are configuring more than one worker change the hostname for each of them

```bash
sudo hostnamectl set-hostname "w1"
exec bash
```

Add the following entries in /etc/hosts file on each node (Replace IP Addresses With Your Public/Private IPs)

```bash
sudo vi /etc/hosts
```

```bash
MASTER_IP   master
WORKER1_IP  w1
```

Real Environment >> Disable swap & Add kernel Parameters

Execute beneath swapoff and sed command to disable swap. Make sure to run the following commands on all the nodes.

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

Load the following kernel modules on all the nodes

```bash
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```

```bash
tail /etc/modules-load.d/containerd.conf
sudo modprobe overlay
sudo modprobe br_netfilter
```

Set the following Kernel parameters for Kubernetes run beneath tee command

```bash
sudo tee /etc/sysctl.d/kubernetes.conf <<EOT
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOT
```

Reload the above changes run

```bash
sudo sysctl --system
```

Install Containerd Runtime

In this guide we are using containerd runtime for our Kubernetes cluster. So, to install containerd first install its dependencies.

```bash
sudo apt update -y
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```

Enable docker repository

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

Now run following apt command to install containerd

```bash
sudo apt update
sudo apt install -y containerd.io
```

Configure containerd so that it starts using systemd as cgroup.

```bash
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```

Restart and enable containerd service

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

```bash
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

Next, run following echo command to add Kubernetes apt repository.

```bash
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

Install Kubectl, Kubeadm and Kubelet

Post adding the repositories, install Kubernetes components like kubectl, kubelet and Kubeadm utility on all the nodes. Execute following set of commands

```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Install Kubernetes Cluster on Ubuntu 20

Now, we are all set to initialize Kubernetes cluster. Run the following Kubeadm command on the master node only.

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

If any error >> bypass >>

```bash
sudo kubeadm init --ignore-preflight-errors=NumCPU,Mem,Swap --pod-network-cidr=10.244.0.0/16
```

If any problem... Reset and start again.

```bash
sudo kubeadm reset
```

After the initialization is complete, you will see a message with instructions on how to join worker nodes to the cluster. Make a note of the kubeadm join command for future reference.

So, to start interacting with cluster, run following commands on the master node

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

next, try to run following kubectl commands to view cluster and node status

```bash
kubectl cluster-info
kubectl get nodes
```

If any problem to check errors >>>

```bash
kubectl cluster-info dump
```

Join Worker Nodes to the Cluster

On each worker node, use the kubeadm join command you noted down earlier after initializing the master node. It should look something like this:

Example >> 

```bash
sudo kubeadm join MASTER_IP --token TOKEN \
   --discovery-token-ca-cert-hash sha256:HASH
```

Check the nodes status from master node using kubectl command

```bash
kubectl get nodes
```

As we can see nodes status is ‘NotReady’, so to make it active. We must install CNI (Container Network Interface) or network add-on plugins like Calico, Flannel and Weave-net.

Install Calico Network Plugin

A network plugin is required to enable communication between pods in the cluster. Run following kubectl command to install Calico network plugin from the master node

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml
```

Verify the status of pods in kube-system namespace

```bash
kubectl get pods -n kube-system
```

Perfect, check the nodes status as well

```bash
kubectl get nodes
```

Great, above confirms that nodes are active node. Now, we can say that our Kubernetes cluster is functional.

In your lab or not production server deployment >> if not enough memory >> after installation enable swap or config it >> Swap Config

```bash
sudo fallocate -l 4G /swapfile
ls -anp /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**********

Test Your Kubernetes Cluster Installation

```bash

kubectl apply -f https://k8s.io/examples/service/load-balancer-example.yaml

kubectl get deployments hello-world

kubectl describe deployments hello-world

kubectl get replicasets

kubectl expose deployment hello-world --type=LoadBalancer --name=my-service

kubectl get services my-service

kubectl describe services my-service

kubectl get pods --output=wide

kubectl get pods --all-namespaces

kubectl get services

kubectl patch svc <svc-name> -n <namespace> -p '{"spec": {"type": "LoadBalancer", "externalIPs":["172.31.71.218"]}}'

kubectl patch svc my-service -n default -p '{"spec": {"type": "LoadBalancer", "externalIPs":["193.176.31.120"]}}'

kubectl delete services my-service

kubectl delete deployment hello-world

```


**********

Helm Installation on Ubuntu 20 or 22 and Test

Solution 1:

```bash

sudo snap install helm --classic

```

Solution 2:

```bash

curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null

sudo apt-get install apt-transport-https --yes

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list

sudo apt-get update

sudo apt-get install helm

```

Helm Example >> Apache

URL >> https://bitnami.com/stack/apache/helm

```bash
helm install my-release oci://registry-1.docker.io/bitnamicharts/apache
```

Please be patient while the chart is being deployed

Get the Apache URL by running:

Please ensure an external IP is associated to the my-release-apache service before proceeding

Watch the status using: 

```bash

kubectl get svc --namespace default -w my-release-apache

export SERVICE_IP=$(kubectl get svc --namespace default my-release-apache --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")

echo URL            : http://$SERVICE_IP/

```

WARNING: You did not provide a custom web application. Apache will be deployed with a default page. Check the README section "Deploying your custom web application" in https://github.com/bitnami/charts/blob/main/bitnami/apache/README.md#deploying-a-custom-web-application.

```bash

kubectl patch svc my-release-apache -n default -p '{"spec": {"type": "LoadBalancer", "externalIPs":["148.251.119.125"]}}'

kubectl get pods

kubectl get nodes

kubectl get services

kubectl delete deployment my-release-apache

kubectl delete service my-release-apache

```


**********

Helm Example >> Nginx

URL >> https://artifacthub.io/packages/helm/bitnami/nginx

```bash

helm install my-nginx oci://registry-1.docker.io/bitnamicharts/nginx

kubectl patch svc my-nginx -n default -p '{"spec": {"type": "LoadBalancer", "externalIPs":["148.251.119.125"]}}'

kubectl get pods

kubectl get nodes

kubectl get services

kubectl delete deployment my-nginx

kubectl delete service my-nginx

```


