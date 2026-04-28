### General Policies
https://github.com/ahmetb/kubernetes-network-policy-recipes

```
cd ~
mkdir netpol && cd netpol
```
### Egress Policy
Create Pods for the Policy
```
cat << 'EOF' > 01_egress_pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: netpol-pod1
spec:
  hostname: pod1
  containers:
  - name: nginx-con
    image: nginx:1.23.1
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: netpol-pod2
spec:
  hostname: pod2
  containers:
  - name: nginx-con
    image: nginx:1.23.1
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: netpol-pod3
spec:
  hostname: pod3
  containers:
  - name: nginx-con
    image: nginx:1.23.1
    ports:
    - containerPort: 80
EOF
```

``` bash
kubectl apply -f 01_egress_pod.yaml
```

#### Network policy

Create a NetworkPolicy

```
cat << 'EOF' > 02_egress.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      netpol: eg
  policyTypes:
    - Egress
  egress:
    - to:
        #- namespaceSelector:
        #    matchLabels:
        #      allow: ns
        - podSelector:
            matchLabels:
              allow: eg
      ports:
        - protocol: TCP
          port: 90
EOF
```

``` bash
kubectl apply -f 02_egress.yaml
```

#### Test Connection

Get IP of the Pods

``` bash
kubectl get pod netpol-pod1 --template '{{.status.podIP}}'
echo ""
kubectl get pod netpol-pod2 --template '{{.status.podIP}}'
echo ""
kubectl get pod netpol-pod3 --template '{{.status.podIP}}'
echo ""
```


Connect to Pod 1
``` bash
kubectl exec -it netpol-pod1 -- bash
```

Look at resolve.conf
```
cat /etc/resolv.conf
```

Access the Pods via its hostnames
```
curl pod1
curl pod2
curl pod3
```

Install Ping
``` bash
apt update
``` 

``` bash
apt install iputils-ping -y
```


Try to ping the hosts
``` bash
ping pod1
ping pod2
ping pod3
```

## Connect Network Policy with Pod

  
Add labels to the created Pods
```
cat << 'EOF' > 01_egress_pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: netpol-pod1
  labels:
    netpol: eg
spec:
  hostname: pod1
  containers:
  - name: nginx-con
    image: nginx:1.23.1
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: netpol-pod2
  labels:
    netpol: eg
spec:
  hostname: pod2
  containers:
  - name: nginx-con
    image: nginx:1.23.1
    ports:
    - containerPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: netpol-pod3
  labels:
    netpol: eg
spec:
  hostname: pod3
  containers:
  - name: nginx-con
    image: nginx:1.23.1
    ports:
    - containerPort: 80
EOF
```

``` bash
kubectl apply -f 01_egress_pod.yaml
```

```
kubectl get pod --show-labels
```


#### No Connection

If we connect to a Pod and try to access the others we will get a timeout

```
kubectl exec -it netpol-pod1 -- bash
```

``` bash
ping pod2
ping pod3
```

  
#### Allow Egress

```
cat << 'EOF' > 03_egress.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress
  namespace: default
spec:
  podSelector:
    matchLabels:
      netpol: eg
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 10.100.0.0/16
      ports:
        - protocol: TCP
          port: 80
EOF
```

``` bash
kubectl apply -f 03_egress.yaml
```

Connections with the IP work

``` bash
kubectl get pod netpol-pod1 --template '{{.status.podIP}}'
echo ""
kubectl get pod netpol-pod2 --template '{{.status.podIP}}'
echo ""
kubectl get pod netpol-pod3 --template '{{.status.podIP}}'
echo ""
```

```
kubectl exec netpol-pod1 -- curl 10.100.167.141
kubectl exec netpol-pod1 -- curl 10.100.167.140
kubectl exec netpol-pod1 -- curl 10.100.167.142
```

If we try the DNS name it wont work.

```
kubectl exec netpol-pod1 -- curl pod1
kubectl exec netpol-pod1 -- curl pod2
kubectl exec netpol-pod1 -- curl pod3
```


Allow based on label

```
cat << 'EOF' > 04_egress.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-label
  namespace: default
spec:
  podSelector:
    matchLabels:
      netpol: eg
  policyTypes:
    - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          netpol: eg
    ports:
    - protocol: TCP
      port: 80
EOF
```

``` bash
kubectl delete -f 03_egress.yaml
kubectl apply -f 04_egress.yaml
```

```
kubectl exec netpol-pod1 -- curl 10.100.167.141
kubectl exec netpol-pod1 -- curl 10.100.167.140
kubectl exec netpol-pod1 -- curl 10.100.167.142
```

