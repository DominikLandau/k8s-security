### How do the internal components in Kubernetes communicate

In Kubernetes each Component gets its own "kubeconfig" which allows the communication with the API Server.

Analyse the different configs
```
sudo cat /etc/kubernetes/controller-manager.conf
```

```
sudo cat /etc/kubernetes/admin.conf
```

With this command we can extract the certificate of one of the conf file, base64 decode it and save it to a file
```
sudo cat scheduler.conf | grep "client-certificate-data" | cut -d ' ' -f 6 | base64 -d | sudo tee scheduler.cr
```

If we now look at the certificate we can see under the subject the user of the cert.
```
openssl x509 -noout -text -in scheduler.crt
```

