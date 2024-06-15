# Installing Kubernetes Using Kubeadm

- Please follow the below instruction one by one to install Kubernetes using Kubeadm

## Run on Both Master & Worker Nodes

- Switch to root user
```
sudo su
```
- Turn Off Swap Space
```
swapoff -a
sed -i '/ swap / s/^\.(.*\)$/#\1/g' /etc/fstab
```
- Install Packages to install K8S and containerd
```
apt update -y
apt install -y apt-transport-https -y
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF

apt update -y
apt install -y kubelet kubeadm containerd kubectl
```
- Hold the pacakges from being automatically upgrade
```
apt-mark hold kubelet kubeadm kubectl containerd
```

- Configure containerd & load the necessary modules for containerd
```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```
```
modprobe overlay
modprobe br_netfilder
```
- Setup the required Kernel Parameters
```
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sysctl --system
```

- Configre Containerd
```
mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
systemctl restart containerd
```

- Start & Enable Kubelet Service
```
systemctl deamon-reload
systemctl start kubelet
systemctl enable kubelet.service
```

## Run only on Master Node

- Swith to Root User
```
sudo su
```
- Initialize K8S master
```
kubeadm init
```

- Exist as a root user and run the followiing commands
```
exit

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
- Verify Kubectl
```
kubectl get pods -n kube-system
```

- To run networking pod we will be using Weave as a networking support
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

kubectl get nodes
kubectl get pods --all-namespaces
```

- Now we have to make connectivity between Master nodes & Worker Nodes. 
- To do that we have to join worker nodes to the master node. For that we need to generate token. Type the below command to get the token
```
kubeadm token create --print-join-command
```

- Now you will get the output of the above command, copy and past the output on both the worker nodes as a root user
- Now run the below command and see if you were able to see the worker node details in master or not
```
kubectl get nodes
```
- Out put should show as below if you were able to connect both worker and master nodes
```
NAME             STATUS     ROLES                   AGE  VERSION
<ip-address>     Ready      control-plane,master    20m  v1.23.3
<ip-address>     NotREady   <none>                  12s  v1.23.3
<ip-address>     NotREady   <none>                  3s  v1.23.3
```