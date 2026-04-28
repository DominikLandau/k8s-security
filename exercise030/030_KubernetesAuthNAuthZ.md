### Kubernetes User
Kubernetes has no direct user management. For its internal communication it uses x509 certificates which are signed by the Cluster CA.

https://kubernetes.io/docs/reference/access-authn-authz/authentication/

Cluster CA
```
cat /etc/kubernetes/pki
```

To get user in Kubernetes we typically use an Identity Provider (IdP). All the managed Kubernetes solutions provide this integration directly and configure everything automatically.

If we want to set it up ourselves we need to get an IdP and set the right parameter on the API Server

https://kubernetes.io/docs/reference/access-authn-authz/authentication/#configuring-the-api-server

IdP: https://github.com/keycloak/keycloak