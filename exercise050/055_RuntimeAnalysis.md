### Falco
https://github.com/falcosecurity/falco

Install everything on a Kind Cluster

Install Helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4  
chmod 700 get_helm.sh  
./get_helm.sh
```

Add the Falco repo
```
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
```

Install Falco with UI
```
helm install --namespace falco falco falcosecurity/falco \
  --set falcosidekick.enabled=true \
  --set falcosidekick.webui.enabled=true \
  --create-namespace
```

```
watch kubectl get pod -n falco
```

Get the Falco UI service
```
kubectl get svc -n falco falco-falcosidekick-ui
```

Change the ClusterIP to NodePort
```
kubectl get svc -n falco falco-falcosidekick-ui -o yaml > falco.yaml
nano falco.yaml
kubectl apply -f falco.yaml 
```

Use the Node ip and Node port to connect to the Falco UI (Login admin/admin)
```
kubectl get svc -n falco falco-falcosidekick-ui
kubectl get node -o wide
```

Trigger Falco
```
kubectl run mypod --image=nginx
kubectl exec mypod -- cat /etc/shadow
```
### Tracee
https://github.com/aquasecurity/tracee

https://aquasecurity.github.io/tracee/v0.9/getting-started/kubernetes-quickstart/
```
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/tracee/main/deploy/kubernetes/tracee/tracee.yaml
```

### Trivy Operator
https://github.com/aquasecurity/trivy-operator