### Admission Controller
Admission control is a central part of Kubernetes. There are already some admission controller in place which modify the behaviour of Kubernetes. One of the is for example the LimitRange

Kubernetes offers two ways interacting with the admission process.
- Webhooks (old way)
- Policy (new way)

### Validating Admission Policy

These are available directly with the Kubernetes installation. This policy requires a label "validating: isenabled".
```
apiVersion: v1
kind: Namespace
metadata:
  name: validatingns
---
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingAdmissionPolicy
metadata:
  name: require-validating-label
spec:
  failurePolicy: Fail
  matchConstraints:
    resourceRules:
    - apiGroups: [""]
      apiVersions: ["v1"]
      operations: ["CREATE"]
      resources: ["pods"]
  validations:
  - expression: "has(object.metadata.labels) && object.metadata.labels['validating'] == 'isenabled'"
    message: "Pods in validatingns must have label validating=isenabled"
---
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingAdmissionPolicyBinding
metadata:
  name: require-validating-label-binding
spec:
  policyName: require-validating-label
  validationActions: [Deny]
  matchResources:
    namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: validatingns
```


```
kubectl get validatingadmissionpolicies,validatingadmissionpolicybindings
```

If we create a Pod with the following spec we will get an error.
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: testvalidation
  name: testvalidation
  namespace: validatingns
spec:
  containers:
  - image: nginx
    name: testvalidation
```

Adding the "validating" label with the wrong key gives us also a warning.
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: testvalidation
    validating: doIneedThis
  name: testvalidation
  namespace: validatingns
spec:
  containers:
  - image: nginx
    name: testvalidation
```

So if we change label to the correct one we will be able to create the Pod.
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: testvalidation
    validating: isenabled
  name: testvalidation
  namespace: validatingns
spec:
  containers:
  - image: nginx
    name: testvalidation
```


### Mutating Admission Policy
To activate the mutating admission policy we need to add some additional parameter to the API Server or use the Kind Cluster from exercise 011.
```
- --feature-gates=MutatingAdmissionPolicy=true
- --runtime-config=admissionregistration.k8s.io/v1beta1=true
```

Here we create a policy which addes a label to a pod
```
apiVersion: v1
kind: Namespace
metadata:
  name: mutatingns
---
apiVersion: admissionregistration.k8s.io/v1beta1
kind: MutatingAdmissionPolicy
metadata:
  name: add-mutating-label
spec:
  failurePolicy: Fail
  reinvocationPolicy: IfNeeded
  matchConstraints:
    resourceRules:
    - apiGroups: [""]
      apiVersions: ["v1"]
      operations: ["CREATE"]
      resources: ["pods"]
  mutations:
  - patchType: ApplyConfiguration
    applyConfiguration:
      expression: >
        Object{
          metadata: Object.metadata{
            labels: {
              "mutating": "added"
            }
          }
        }
---
apiVersion: admissionregistration.k8s.io/v1beta1
kind: MutatingAdmissionPolicyBinding
metadata:
  name: add-mutating-label-binding
spec:
  policyName: add-mutating-label
  matchResources:
    namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: mutatingns
```

The Pod creation works without a problem and we get automatically a label added.
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: testmutating
  name: testmutating
  namespace: mutatingns
spec:
  containers:
  - image: nginx
    name: testvalidation
```

```
kubectl get pod -n mutatingns testmutating -o yaml | head -n 15
```