### Kubernetes Authentification using x509 Certificates

Caution: You should not create user with this method, because the users will be valid as long as the Cluster Certificate is valid and you have no possibility of disabling the created users. If you want to use user, use an external OIDC compliant identity provider.

```
mkdir user && cd user
```
#### Generate Keys
Generate a rsa <b>private key</b>

``` bash
openssl genrsa -out kube.key 2048
```

Generate a certificate signing request. The <b>CN</b> will be the name for the user in the Cluster.

``` bash
openssl req -new -key kube.key -subj "/CN=kube" -out kube.csr
```

Encode the certificate signing request in base64 and remove the newline characters for easy copying. <b>Copy the base64 output</b>.

``` bash
cat kube.csr | base64 | tr -d '\n'
```

## Create CertificateSigningRequest

Create a <b>CertificateSigningRequest</b>. This will associate the private key of the user "kube" with the Kubernetes Cluster.

```
nano csr.yaml
```

``` yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: kube
spec:
  groups:
  - system:authenticated
  usages:
  - digital signature
  - key encipherment
  - client auth
  signerName: kubernetes.io/kube-apiserver-client
  request: <KUBE.CSR IN BASE64> # Paste from the command above
```

Make sure to paste the output from the <b>cat kube.csr | base64 | tr -d '\n'</b> command in the <b>request</b> field.

Apply the <b>CertificateSigningRequest</b>

``` bash
kubectl apply -f csr.yaml
```

Verify the creation. You can use <b>csr</b> instead of <b>certificatesigningrequest</b>. Notice that the request is <b>Pending</b>. The Cluster Admin has to approve to before it becomes active.

``` bash
kubectl get certificatesigningrequest
```

Approve the Certificate Signing Request for user "kube". Kubernetes now signs the certificate using the CA key-pair and generates a certificate for the user.

``` bash
kubectl certificate approve kube
```

Verify the CSR is now <b>Approved, Issued</b>.

``` bash
kubectl get certificatesigningrequest
```

Approved CSR are deleted automatically after 1 hour but you can also delete it right now.

Now that the certificate has been approved and signed by the Kubernetes CA, you need to get the signed certificate from the cluster. Since it was approved, the signed certificate is now available on the Kubernetes object in the <b>.status.certificate</b> field. Retrieve this value and encode it in base64.

``` bash
kubectl get csr kube -o jsonpath='{.status.certificate}' | base64 -d > kube.crt
```

The <b>kube.crt</b> file is the signed client certificate which is used to authenticate the user <b>kube</b>. The <b>kube.key</b> which you created at the beginning it the private key. As long as you have these two files, you can get authenticated with the cluster.

### Configure KubeConfig

Add the user <b>kube</b> into your <b>.kube/config</b> file.

``` bash
kubectl config set-credentials kube --client-certificate=kube.crt --client-key=kube.key
```

Or with the content of the files present in the kubeconfig
``` bash
kubectl config set-credentials kube --client-certificate=kube.crt --client-key=kube.key --embed-certs=true
```

Verify the user <b>kube</b> was added.

``` bash
kubectl config view
```

List the users
```
kubectl config get-users
```

List the cluster
```
kubectl config get-clusters
```

Create a new context for the Cluster
```
kubectl config set-context kubecontext --user kube --cluster kubernetes
```

List all contexts
```
kubectl config get-contexts
```

Switch between contexts
```
kubectl config set-context --context kubecontext --current
kubectl config set-context --context kubernetes-admin@kubernetes --current
```

- Instead of using
	- <b>client-certificate</b> and <b>client-key</b> to reference the files you can also use
	- <b>client-certificate-data</b> and <b>client-key-data</b> to paste the contents of the files encoded in base64. This is helpful when creating a "new" <b>kubeconfig</b> file which will be distributed to the user. This file should of course only include the credentials for the user and not for the admin. (--embed-certs=true)

Check which users have permissions for which resources

``` bash
# Output: "yes", since you are the cluster admin
kubectl auth can-i create pods

# Output: "no", since "kube" is authenticated but not authorized
kubectl auth can-i create pods --as kube
```

