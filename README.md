## K8S-Playground

Vagrant is used to deploy Ubuntu VMs on Apple silicon (Macbook Air 2020 M1)

### Step 1 : Install vagrant.
```shell
brew install vagrant
```

### Step 2: Install vmware_desktop
```shell
vagrant plugin install vagrant-vmware-desktop
```

### Step 3: Create 3 VMs (1 KubeMaster, 2 workers) using the Vagrantfile
```shell
vagrant up --provider vmware_desktop
```
The vagrant file provisions the kubemaster with IP 192.168.56.2, and the workers with 192.168.56.3 and 192.168.56.4
Hostname of the master is kubemaster, and of the workers is kubenode01/02

Status of the VMs can be checked with the command:
```shell
vagrant status
```

### Step 4: Logon to the VMs by issuing the following command (for ex for the master node)
```shell
vagrant ssh kubemaster
```

## Step 5: Install all the binaries and create a cluster

#####On all 3 nodes, run the following commands
```shell
#Disable the firewall
sudo ufw disable

#Disable swap, otherwise kubectl won't work
sudo  swapoff -a; sed -i '/swap/d' /etc/fstab

#Let kubernetes see bridged-traffic
sudo cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

#Install docker engine
sudo apt-get update
sudo apt install docker.io

#Create a kubernetes repo and install kubernetes components
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

# Update /etc/hosts on all nodes
cat >> /etc/hosts <<EOF
192.168.56.2  kubemaster
192.168.56.3  kubenode01
192.168.56.4  kubenode02

EOF

```

##### Run the following on only the master node
```shell
sudo kubeadm init --apiserver-advertise-address=192.168.56.2 --pod-network-cidr=192.168.0.0/16
##Considering that 192.168.56.2 is master node

#Install pod network add on
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

#Print the join command that needs to be run on the worker nodes
sudo kubeadm token create --print-join-command

# Copy the kube config file into the user's home dir to be able to run kubectl commands. 
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


```

##### Run the following on the worker nodes only
```shell
sudo kubeadm join.......   ## Take the join command from above

# Copy the content of /etc/kubernetes/admin.conf file from the master node, to you user's home dir at the path $HOME/.kube/config. Create the .kube dir mkdir -p $HOME/.kube
# Now copy the file /etc/kubernetes/admin.conf from the master node to the $HOME/.kube/config file. This will allow this user on worker to run kubectl on the cluster

```




