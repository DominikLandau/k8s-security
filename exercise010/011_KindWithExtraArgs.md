### Extra args

It is possible to customize the Kind installation in different ways. Here we activate some additional Feature Gates, so that we can use the "MutatingAdmissionPolicy".
```
cat << 'EOF' > c2.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: kind2
featureGates:
  MutatingAdmissionPolicy: true
runtimeConfig:
  admissionregistration.k8s.io/v1beta1: true
nodes:
- role: control-plane
  image: kindest/node:v1.35.1
- role: worker  
  image: kindest/node:v1.35.1
EOF
```

```
kind create cluster --config c2.yaml
```

```
kind delete clusters kind2
```