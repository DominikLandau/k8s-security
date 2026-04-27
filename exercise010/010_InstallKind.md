### Installation Server C
### Install Docker
Depending on the OS install Docker
https://docs.docker.com/engine/install/

Here is the installation steps for debian systems
```
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

```
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

After installing Docker add yourself to the docker group
```
sudo usermod -aG docker $USER
```

Because of the Guacamole we need to do a reboot to add ourselfs to the docker group
```
sudo reboot now
```

Now Docker works
```
docker info
```

### Install Kubectl
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
### Install Kind
https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries
```
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.31.0/kind-linux-amd64
# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.31.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

Increase system parameter, so that multiple Kind instances can run
```
echo fs.inotify.max_user_watches=655360 | sudo tee -a /etc/sysctl.conf  
echo fs.inotify.max_user_instances=1280 | sudo tee -a /etc/sysctl.conf  
sudo sysctl -p
```
### Create a Kind Cluster
Simple config file for a Kind cluster
```
cat << 'EOF' > c1.yaml
kind: Cluster  
apiVersion: kind.x-k8s.io/v1alpha4  
name: kind1  
nodes:  
- role: control-plane  
  image: kindest/node:v1.35.1 
- role: worker  
  image: kindest/node:v1.35.1
EOF
```

```
kind create cluster --config c1.yaml
```

By default the kubeconfig of the last Kind cluster will be active
```
kubectl get node
```

With the typical kubectl commands we can switch between the different contexts
```
kubectl config get-contexts
kubectl config use-context kind-kind1
```

List all available Kind Cluster
```
kind get clusters
```

If needed the Cluster can be deleted
```
kind delete clusters kind1
```
### Access the "Control Plane"

Kind runs the Kubernetes Nodes as Docker container, so we can also directly access these via "docker exec"
```
docker exec -it kind1-control-plane bash
```

In the Docker container show the API Server config
```
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

