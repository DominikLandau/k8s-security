### CIS Benchmarks
https://downloads.cisecurity.org/#/

There are different tools which allow us to check if the created Kubernetes Cluster is setup properly.

### Kube-Bench
https://github.com/aquasecurity/kube-bench

Install the kube-bench tool
```
curl -LO https://github.com/aquasecurity/kube-bench/releases/download/v0.15.0/kube-bench_0.15.0_linux_amd64.deb

sudo dpkg -i kube-bench_0.15.0_linux_amd64.deb 
```

Here we can check a single point from the CIS Benchmarks
```
kube-bench run --check="1.1.2"
```

Here we do a general run
```
kube-bench run
```

Here we check the nodes
```
kube-bench run --targets node
```

Here we check the Control Plane / Master Node
```
kube-bench run --targets master
```

Kube-Bench installs different configs which we can check the Cluster against
```
ls /etc/kube-bench/cfg/
kube-bench run --benchmark cis-1.24
```

### kubescape
https://github.com/kubescape/kubescape

Install kubescape
```
curl -s https://raw.githubusercontent.com/kubescape/kubescape/master/install.sh | /bin/bash
```

```
export PATH=$PATH:/home/kurs/.kubescape/bin
```

https://github.com/kubescape/kubescape#scan-a-running-cluster

Like with Kube-Bench kubescape also provides different rulesets we can check the Cluster against.
```
kubescape  list frameworks
kubescape scan framework cis-v1.12.0
```

### Trivy
https://github.com/aquasecurity/trivy

Install Trivy
```
curl -LO https://github.com/aquasecurity/trivy/releases/download/v0.70.0/trivy_0.70.0_Linux-64bit.deb
```

```
sudo dpkg -i trivy_0.70.0_Linux-64bit.deb
```

Here we do a simple Trivy scan for misconfiguration
```
trivy k8s --report summary --scanners misconfig --compliance=k8s-cis-1.23 --disable-node-collector
```

Like the others Trivy also has some different rulesets
```
Compliance
- k8s-nsa-1.0
- k8s-cis-1.23
- eks-cis-1.4
- rke2-cis-1.24
- k8s-pss-baseline-0.1
- k8s-pss-restricted-0.1
```

Check against CIS
```
trivy k8s --report summary --compliance=k8s-cis-1.23 --disable-node-collector
```

Make a full scan with an output file
```
trivy k8s --report all --compliance=k8s-cis-1.23 --disable-node-collector --format json --output cluster-scan.json
```

```
cat cluster-scan.json | less
```