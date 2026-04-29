### ServiceAccounts
Each Namespace has the default ServiceAccount available which will be added to a Pod by default if not specified different.

```
kubectl get sa
```

Additionally to the default ServiceAccount, in the kube-system Namespace there are a lot more. These ServiceAccounts are associated to the different controller in the Controller Manager.

```
kubectl get sa -n kube-system
```

Controller in Kubernetes
https://github.com/kubernetes/kubernetes/blob/master/cmd/kube-controller-manager/names/controller_names.go

```
kubectl get clusterrolebindings.rbac.authorization.k8s.io | grep controller
```

```
kubectl get clusterrole.rbac.authorization.k8s.io | grep controller
```

### Manage ServiceAccounts

Create a new ServiceAccount
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: mysa1
```

Create a ServiceAccount without automounting the token

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: mysa2
automountServiceAccountToken: false
```

 Create a ServiceAccount with a long lived token
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: mysa3
---
apiVersion: v1
kind: Secret
metadata:
  name: mys3-token
  annotations:
    kubernetes.io/service-account.name: mysa3
type: kubernetes.io/service-account-token
```


#### Create a token for a ServiceAccount
```
kubectl create token mysa | tee mytoken
```

Create a nginx Pod
```
kubectl run testpod --image=nginx
```

Test accessing the Kubernetes API Server via the Pod.
```
kubectl exec testpod -- curl https://kubernetes -k
```

Access the API Server via the Pod and use the token for the Service Account which was created earlier.
```
kubectl exec testpod -- curl --header "Authorization: Bearer $(cat mytoken)" https://kubernetes -k
```
```
curl --header "Authorization: Bearer $mytoken" https://kubernetes -k
```

General way to access Kubernetes API via the ServiceAccount
```
# Define variables
APISERVER=https://kubernetes.default.svc
SERVICEACCOUNT=/var/run/secrets/kubernetes.io/serviceaccount
NAMESPACE=$(cat ${SERVICEACCOUNT}/namespace)
TOKEN=$(cat ${SERVICEACCOUNT}/token)
CACERT=${SERVICEACCOUNT}/ca.crt

# Example: List pods in the current namespace
curl --cacert ${CACERT} --header "Authorization: Bearer ${TOKEN}" -X GET ${APISERVER}/api/v1/namespaces/${NAMESPACE}/pods   
```

### Create (Cluster)Roles
https://kubernetes.io/docs/reference/access-authn-authz/rbac/#command-line-utilities

```
kubectl create clusterrole myrole --verb get --resource pod --resource deployment -o yaml --dry-run=client
```

```
kubectl create clusterrolebinding mysa1-myrole --serviceaccount default:mysa1 --clusterrole myrole -o yaml --dry-run=client
```

#### Get all verbs
```
kubectl api-resources -o wide
```

#### Test if SA can list Pods
```
kubectl auth can-i <verb> <resource> --as=system:serviceaccount:<namespace>:<serviceaccountname>
```

```
kubectl auth can-i list pods --as=system:serviceaccount:default:mysa1
```

=> No

```
kubectl auth can-i get pods --as=system:serviceaccount:default:mysa1
```

=> Yes

### Assign a ServiceAccount to a Pod

```
kubectl run mydefaultpod --image=nginx
```

Even though we didn't define any ServiceAccount when creating the Pod, a ServiceAccount was added to the Pod
```
kubectl describe pod mydefaultpod | head -n 5
```

Additionally to that a volume with information about the ServiceAccount was also added
```
kubectl get pod mydefaultpod -o yaml | grep volumeMounts: -A 3
```

Three files are available
```
kubectl exec mydefaultpod -- ls /var/run/secrets/kubernetes.io/serviceaccount
```

To disallow the mounting of ServiceAccount we can use the label automountServiceAccountToken set to false either on the ServiceAccount object or the Pod itself

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: mysa2
automountServiceAccountToken: false
```

```
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  serviceAccountName: mysa2
  automountServiceAccountToken: false
  containers:
  - name: app
    image: nginx
```