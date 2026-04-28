### Gateway API
The Gateway API is the next iteration of the Kubernetes Ingress. While the Ingress was not that deeply specified all the different vendors used different implementation mostly with custom annotations.

To create a new standard the Gateway API project was created
https://gateway-api.sigs.k8s.io/reference/spec/

Through the modular approach the Gateway API is more flexible and offers a broader range of security feature. 
https://gateway.envoyproxy.io/docs/concepts/

Envoy Gateway offers here a broad range of different tutorials
https://gateway.envoyproxy.io/docs/tasks/security/backend-mtls/