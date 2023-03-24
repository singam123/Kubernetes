# Kubernetes

## Configuring Kubernetes on Master node 

### Set the hostname as master
```
sudo hostnamectl set-hostname "Master"
```

### turn off the swap 
```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
### Loading kernel modules on Master node
```
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```
### Set the Kernel Parameters
```
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

### Reload sysctl
```
sudo sysctl --system
```
### Installing Containerd run time
```
sudo apt install -y curl software-properties-common apt-transport-https ca-certificates
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y containerd.io
```
### Configure containerd to start using systemd as cgroup.
```
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```
### Restart and enable Containerd Service
```
sudo systemctl restart containerd
sudo systemctl enable containerd
```
### Adding apt repositories of Kubernetes
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

### Installing Kubectl, Kubeadm and kubelet
```
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
### Run Only on *Master node*
```
sudo kubeadm init
```
If Installation is successfull will be able to see similar output as shown in below image

![image](https://user-images.githubusercontent.com/36833442/224369698-ee5ca036-1eed-44aa-9d84-ef0ed68f4e1b.png)

### RUN these command on Master Node to interact with cluster
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### View and verify the cluster and node status using kubectl
```
kubectl cluster-info
kubectl get nodes
```
If the configuration is sucessfull then will be able to see the output as below

![image](https://user-images.githubusercontent.com/36833442/224370698-6f18f9f7-bdd3-46cb-8f9b-e5a1dee4c4ca.png)

As we can see nodes status is ‘NotReady’, so to make it active. We must install CNI (Container Network Interface) or network add-on plugins like Calico, Flannel and Weave-net.

### Installing Calico Pod Network Add-on 
```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```
Output of above command would look like below

![image](https://user-images.githubusercontent.com/36833442/224371550-20669930-8cd8-4564-8f83-870a39181570.png)

### Verify the status of pods in kube-system namespace,
```
kubectl get pods -n kube-system
```
![image](https://user-images.githubusercontent.com/36833442/224371906-735e77a5-cd36-4a02-a1e7-9ea46269199a.png)

### Check the node status 
```
kubectl get nodes
```

## Setting up Worker Node

### Set the hostname as Node1
```
sudo hostnamectl set-hostname "Node1"
```

### turn off the swap 
```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```
### Loading kernel modules on Worker node
```
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```
### Set the Kernel Parameters
```
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

### Reload sysctl
```
sudo sysctl --system
```
### Installing Containerd run time
```
sudo apt install -y curl software-properties-common apt-transport-https ca-certificates
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install -y containerd.io
```
### Configure containerd to start using systemd as cgroup.
```
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
```
### Restart and enable Containerd Service
```
sudo systemctl restart containerd
sudo systemctl enable containerd
```
### Adding apt repositories of Kubernetes
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```

### Installing Kubectl, Kubeadm and kubelet
```
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
### Join the cluster using the token from master node
```
sudo kubeadm join masternodeip:6443 --token <Place your Token > --discovery-token-ca-cert-hash <sha256 hashcode >
```
![image](https://user-images.githubusercontent.com/36833442/224373377-b46f787b-b63c-473a-bdaf-77025b11c26d.png)


## Requirements On Master node --> Open TCP port 6443 both inbound and Outbound to enable communication between cluster and worker nodes

## Incase of any network related issues try loading br_netfilter module

```
sudo modprobe br_netfilter
```
