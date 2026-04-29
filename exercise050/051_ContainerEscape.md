### Analyse the Pods

Create a simple Pod
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: mypod
  name: mypod
spec:
  containers:
  - image: nginx
    name: mypod
```


Switch to server B and analyse the Pod with crictl

```
sudo crictl ps
```


Now inspect the container
```
sudo crictl inspect 6462940a925c8
```


### Create a privileged Pod

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: privpod
  name: privpod
spec:
  containers:
  - image: nginx
    name: mypod
    securityContext:
      privileged: true
```


```
ls /dev/sda*

mkdir /host
mount /dev/sda2 /host

ls /host
```

```
touch /host/fromcontainer.txt
```


