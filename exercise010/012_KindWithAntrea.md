### Change CNI
When installing Kind it will use its default CNI implementation, but it is possible to use other CNIs for testing specific CRDs.

Here we will use Antrea
Antrea: https://antrea.io/docs/v2.6.1/docs/kind/


Check if Open vSwitch is installed

```
sudo aptitude -y install openvswitch-switch
```

```
sudo lsmod | grep open
```


Create a Cluster config and disable the CNI installation.
```
cat << 'EOF' > c3.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: kind3
networking: 
  disableDefaultCNI: true
  podSubnet: 10.10.0.0/16
nodes:
- role: control-plane
  image: kindest/node:v1.35.1
- role: worker  
  image: kindest/node:v1.35.1
EOF
```

```
kind create cluster --config c3.yaml
```

Now we have a Cluster without any CNI, so the CoreDNS Pods wont get any IP and stay in the Pending state
```
kubectl get pod -A
```


Installing Antrea in the easy way with just the YAML files. For HA installations and more customization use Helm install.
```
kubectl apply -f https://github.com/antrea-io/antrea/releases/download/v2.6.1/antrea.yml
```

It will take some time till all Antrea Pods are up and running

Now we have all the CRDs from Antrea available to test

```
kubectl api-resources | grep antrea
```


Antrea Network Policy: https://antrea.io/docs/v2.6.1/docs/antrea-network-policy/
### Delete the Cluster (if needed)
```
kind delete clusters kind3
```