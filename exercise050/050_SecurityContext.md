### Security Context
The Security Context in Kubernetes can be applied to all the Container in a Pod or to a single Container.
https://kubernetes.io/docs/tasks/configure-pod-container/security-context/


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  # Applied to all container
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    supplementalGroups: [4000]
  containers:
  - name: sec-ctx-demo
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    # Only applied to the container
    securityContext:
      allowPrivilegeEscalation: false
```

Reference for the Security Context at the Pod level
https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#security-context

Reference for the Security Context at the Container level
https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#security-context-1

### Set the user for a Pod
With the security context it is possible to set the user for the Pod. In the end this reduces the attack surface, because the container won't run as a privileged user. But now we can run into the Problem, that the Pod won't work because of missing permissions.

In the official Dockerfile of nginx a user called nginx with the uid of 101 is created
https://github.com/nginx/docker-nginx/blob/71081b25390771f6b1275ddf7c73c965f304493f/mainline/debian/Dockerfile#L20

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-test
spec:
  containers:
  - name: nginx
    image: nginx:1.29
    securityContext:
      allowPrivilegeEscalation: false
      runAsUser: 101
```

If we try to start the Pod above we will run into an error
```
kubectl logs nginx-test
```

```
=> [emerg] mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
```

Like we can see here permissions are missing. But this error is only a small part of the problem. Even when the user has access to this path we would still run into a problem, because by default it uses port 80 which is a privileged port.

### Rootless container
So for this there is explicitly the nginx rootless container
https://hub.docker.com/r/nginxinc/nginx-unprivileged/