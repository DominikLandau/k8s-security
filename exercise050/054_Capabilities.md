### Linux Capabilities
https://www.man7.org/linux/man-pages/man7/capabilities.7.html

Linux divides the privileges which User ID 0 has into different subsets, which are called capabilities. These can be enabled or disabled.

To test the capabilities we create a Pod without any.
```
apiVersion: v1
kind: Pod
metadata:
  name: podwithoutcaps
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "36000"]
    securityContext:
      capabilities:
        drop:
        - ALL
```

Now we can connect to it.
```
kubectl exec -it podwithoutcaps -- sh
```

```
whoami
```
=> root

```
touch test.txt
```

If we try to change the owner of the file we get the output the we are not allowed to do this
```
chown nobody test.txt
```
=> chown: test.txt: Operation not permitted

To list the capabilities of a running Pod use the following command to get the Node on which the Pod runs
```
kubectl get pod -o wide
```

Now connect to the Node and check the Container
```
sudo crictl ps
```

```
sudo crictl inspect <containerid> | grep capabilities -A 4
```

