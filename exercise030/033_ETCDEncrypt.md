### Print Content of ETCD

By default the etcd database is not encrypted at rest. so with some tricks it is possible to get all the data out of it without any credentials. The only thing we need is access to the "db" file.

```
sudo strings /var/lib/etcd/member/snap/db
```

Now create a secret in the ETCD.

```
kubectl create secret generic mysecret --from-literal data=geheim

kubectl get secrets
```


With the following command we can search for the secret and print out its data. Because of the different format the etcd uses for storing the data, the output look strange.
```
sudo strings /var/lib/etcd/member/snap/db | grep mysecret -A 10
```


### Encrypt data at rest
https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/

To encrypt data at rest in the etcd there are different ways. On the one hand you can use some external Key Management System or define a key directly in the config file.

For the KMS setup follow the link
https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/

In our case we do it the simple way with just a password. We use the "aescbc" provider which requires use to use a password with the length of 16, 24 or 32 bytes.

```
echo -n "dasistsehrgeheim" | base64
```

```
=> ZGFzaXN0c2VocmdlaGVpbQ==
```

Now we need to create some files and folder.
```
sudo mkdir /etc/kubernetes/enc
sudo touch /etc/kubernetes/enc/enc.yaml
sudo nano /etc/kubernetes/enc/enc.yaml 
```

With the following config we define that secrets will be encrypted when created.
```
# etc/kubernetes/enc/enc.yaml 
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ZGFzaXN0c2VocmdlaGVpbQ==
      - identity: {}
```

Backup the API Server config
```
sudo cp /etc/kubernetes/manifests/kube-apiserver.yaml .
```

Now we need to add the following parameter to the API Server
```
# Arguments:
--encryption-provider-config=/etc/kubernetes/enc/enc.yaml

# Volumemounts:
- name: enc
  mountPath: /etc/kubernetes/enc
  readonly: true

# Volumes:
volumes:
  - hostPath:
      path: /etc/kubernetes/enc
      type: DirectoryOrCreate
    name: enc

```


Restart the API-Server
```
sudo mv /etc/kubernetes/manifests/kube-apiserver.yaml .
```

```
sudo mv kube-apiserver.yaml /etc/kubernetes/manifests/
```

If everything is setup correctly the API Server should start again. If there is a typo in the config the API Server wont come up and debugging it is always difficult.
```
sudo crictl ps
```


Now we create a new secret
```
kubectl create secret generic mysecret2 --from-literal data=encrypted
```

Here we can see, that the new secret is encrypted, but the old one is still in clear form visible.
```
sudo strings /var/lib/etcd/member/snap/db | grep mysecret -A 10
```

So for encrypting this secret, we need to reapply it.
```
kubectl get secrets mysecret -o yaml > mysecret.yaml

kubectl apply -f mysecret.yaml
```

Now this secret is also encrypted.

