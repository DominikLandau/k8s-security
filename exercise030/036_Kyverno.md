### Kyverno Architecture
https://kyverno.io/docs/introduction/how-kyverno-works/
### Install Kyverno
https://kyverno.io/docs/installation/installation/

To keep it simple, we use the YAML file installation. For a HA setup use the Helm chart.
```
kubectl create -f https://github.com/kyverno/kyverno/releases/download/v1.17.2/install.yaml
```

Now we have some additional CRDs available.
```
kubectl api-resources | grep kyverno
```

```
kubectl get pod -A
```
### Kyverno Policies

There is a big list of predefined policies which can be easily used in the cluster.
https://kyverno.io/policies/

### Allow creation only with label

```
mkdir kyverno && cd kyverno
```

We create a new Namespace for the Pods
```
kubectl create ns mytest
```

First we create a simple policy which checks for a label.
```
cat << 'EOF' > kyverno1.yaml
apiVersion: policies.kyverno.io/v1
kind: NamespacedValidatingPolicy
metadata:
  name: check-labels
  namespace: mytest
spec:
  validationActions:
    - Deny
  matchConstraints:
    resourceRules:
      - apiGroups: ['']
        apiVersions: ['v1']
        operations: [CREATE, UPDATE]
        resources: ['pods']
  validations:
    - message: label 'env' is required
      expression: "'env' in object.metadata.?labels.orValue([])"
EOF
```

```
kubectl apply -f kyverno1.yaml 
```

```
kubectl get namespacedvalidatingpolicies.policies.kyverno.io -A
```

If we try to create a Pod without the label it gets rejected.
```
cat << 'EOF' > kyvernoNotAllowed.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: notallowed
  name: notallowed
  namespace: mytest
spec:
  containers:
  - image: nginx
    name: notallowed
EOF
```

```
kubectl apply -f kyvernoNotAllowed.yaml
```

If we add a label key "env" it will create the Pod

```
cat << 'EOF' > kyvernoAllowed.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: notallowed
    env: test
  name: notallowed
  namespace: mytest
spec:
  containers:
  - image: nginx
    name: notallowed
EOF
```

```
kubectl apply -f kyvernoAllowed.yaml
```

### Delete
```
kubectl delete -f kyvernoAllowed.yaml
```

```
kubectl delete -f https://github.com/kyverno/kyverno/releases/download/v1.17.2/install.yaml
```