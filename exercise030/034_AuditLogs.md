### Audit logs
https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/

Kubernetes provides the possibility to use audit logs, but these need to be configured.

So first we create so files and folder.
```
sudo mkdir /var/log/kubernetes/audit/
sudo mkdir /etc/kubernetes/audit
sudo touch /etc/kubernetes/audit/audit-policy.yaml
sudo nano /etc/kubernetes/audit/audit-policy.yaml
```

The following is a simple audit policy which only collects requests for secret resources.
```
# /etc/kubernetes/audit/audit-policy.yaml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - "RequestReceived"
rules:
  - level: RequestResponse
    resources:
    - group: ""
      resources: ["secrets"]
```


To enable the configuration we need to modify the API Server with the following parameter.

```
# Arguments:
--audit-policy-file=/etc/kubernetes/audit/audit-policy.yaml
--audit-log-path=/var/log/kubernetes/audit/audit.log

# Volumemounts:
- mountPath: /etc/kubernetes/audit/audit-policy.yaml
  name: audit
  readOnly: true
- mountPath: /var/log/kubernetes/audit/
  name: audit-log
  readOnly: false

# Volumes:
- hostPath:
    path: /etc/kubernetes/audit/audit-policy.yaml
    type: File
  name: audit
- hostPath:
    path: /var/log/kubernetes/audit/
    type: DirectoryOrCreate
  name: audit-log
```


Now we restart the API Server
```
sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml .
```

```
sudo mv kube-apiserver.yaml /etc/kubernetes/manifests/
```

If everything was done correctly the API Server should restart and work

```
sudo crictl ps
```


The log file should contain some information from the different components interacting with secrets.
```
sudo cat /var/log/kubernetes/audit/audit.log 
```

We can also create a secret ourselves and see the generated audit log.
```
kubectl create secret generic auditsecret --from-literal data=geheim
```

```
sudo tail -f /var/log/kubernetes/audit/audit.log | jq
```
