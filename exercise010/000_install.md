
### Configure the Control Plane and Worker
We use server A as the Kubernetes Control Plane

First get the IP of the server with
```
ip -br addr
```

They should be in the network range of 10.10.60.0/24.
Here it is 10.10.60.139

##### Turn swap off
```
sudo swapoff -a
sudo sed -ri '/\sswap\s/s/^#?/#/' /etc/fstab
```

#### Configure sysctl
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1
EOF
```

```
sudo sysctl --system
```

#### Install containerd

```
sudo apt install curl -y
```
https://containerd.io/releases/#kubernetes-support

Download containerd
```
cversion=2.2.3

curl -LO https://github.com/containerd/containerd/releases/download/v${cversion}/containerd-${cversion}-linux-amd64.tar.gz

curl -LO https://github.com/containerd/containerd/releases/download/v${cversion}/containerd-${cversion}-linux-amd64.tar.gz.sha256sum

cat containerd-${cversion}-linux-amd64.tar.gz.sha256sum | sha256sum --check
```

Extract the containerd content and save it to /usr/local/bin
```
sudo tar Cxzvf /usr/local containerd-${cversion}-linux-amd64.tar.gz
```

Create a containerd systemd service
```
curl -LO https://raw.githubusercontent.com/containerd/containerd/main/containerd.service

sudo mkdir -p /usr/local/lib/systemd/system/

sudo mv containerd.service /usr/local/lib/systemd/system/containerd.service
```

Activate the new systemd service.

```
sudo systemctl daemon-reload
sudo systemctl enable --now containerd

systemctl status containerd
```

Create a configuration for containerd.
```
sudo mkdir -p /etc/containerd

containerd config default | sudo tee /etc/containerd/config.toml
```

Apply the changes
```
sudo systemctl restart containerd.service

systemctl status containerd
```

```
containerd -v
```

#### Install runc

Requiered version
https://github.com/containerd/containerd/blob/release/2.2/script/setup/runc-version

```
rversion=1.4.2

curl -LO https://github.com/opencontainers/runc/releases/download/v${rversion}/runc.amd64

curl -LO https://github.com/opencontainers/runc/releases/download/v${rversion}/runc.sha256sum

cat runc.sha256sum | grep runc.amd64 | sha256sum --check
```

```
sudo install -m 755 runc.amd64 /usr/local/bin/runc
```

```
runc -v
```

#### Install kubectl, kubeadm and kubelet

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```

```
sudo mkdir -p -m 755 /etc/apt/keyrings

curl -fsSL -k https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

```
sudo apt update
```


```
sudo apt install -y kubectl=1.35* kubeadm=1.35* kubelet=1.35*
```

```
sudo apt-mark hold kubelet kubeadm kubectl
```

```
kubectl version
kubeadm version
kubelet --version
```

### Specific to the Control Plane

```
sudo kubeadm init --pod-network-cidr=10.100.0.0/16
```

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

```
kubectl get node
kubectl get pod -A
```


### Install CNI
https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart

```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.5/manifests/tigera-operator.yaml
```

```
curl -LO https://raw.githubusercontent.com/projectcalico/calico/v3.31.5/manifests/custom-resources.yaml 

sed -i 's/cidr: 192.168.0.0\/16/cidr: 10.100.0.0\/16/' custom-resources.yaml

kubectl apply -f custom-resources.yaml
```

```
kubectl get pod -A
```


### Worker Node
Now switch to server B and jump back to <b>Configure the Control Plane and Worker</b> and do the configuration
- Configure sysctl
- Install Containerd
- Install runc
- Install kubectl, kubelet, kubeadm

On the Control Plane run:
```
kubeadm token create --print-join-command
```


Now join the Worker Node to the Cluster

List all Nodes
```
kubectl get node
```
