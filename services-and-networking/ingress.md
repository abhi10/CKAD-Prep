# H1 Ingress

Ingress helps in users accessing an application Using
externally facing url's that can be configured to route to
different services.
Layer 7 Load Balancer served by K8.
Ingress is not deployed by Default
Still need to expose:
- nodePort
- xternal lB

# H3 Uses of Ingress:
- Authentication
- SSL
- URL based Route Configuration

Ingress Components:
- Ingress Controller
  - GCP - supported
  - nginx - supported
  - Istio
- Ingress Resources

# H3 Ingress Controller
Can monitor Ingress resources if they change.

Deployment Definition:
```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      containers:
      - name: nginx-ingress-controller
        image: quay.io/kubernetes-ingress-controller/
               nginx-ingress-controller:0.21.0

      args:
        - /nginx-ingress-controller
        - --configmap=$(POD_NAMESPACE)/nginx-configuration
      env: # this is where the nginx controller lives and is deployed to.
        - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name

        - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace         

      ports:
        - name: http
          containerPort: 80
        - name: https:
          containerPort: 443    
      // some more data
```
* Service Definition. # to Expose Nginx Controller to External World:
```
apiVersion: extensions/v1beta1
kind: Service
metadata:
  name: nginx-ingress
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress
```
* ConfigMap
```
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
```
* Auth of type ServiceAccount for RBAC/Permissions.
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
```

ConfigMap can contain nginx controller for modular design.

# H3 Ingress Resources:
Route URL Configurations.

Ingress Definition File:
ingress-wear.yaml
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  backend:
    serviceName: wear-service
    servicePort: 80
```
Ingress Resource rules:
Multiple Rules for different path/sections.
- Rules for Each Host.
- Each Rule has multiple Pods to route traffic based on URL.

Ingress Rules
handling traffic to single domain name
ingress-wear-watch.yaml

* ONE RULE TWO PATHS
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - http:
      paths
      - path: /wear  # one path for each URL
      backend:
        serviceName: wear-service
        servicePort: 80
      - path:  /watch
      backend:
        serviceName: watch-service
        servicePort: 80
```
Note: there is a default backend => deploy such a service.

* 2 RULES AND ONE PATH FORMAT:

wear.online.com   <======> watch.online.com
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - host: wear.my-online-store.com
    http:
      paths
      - backend:
          serviceName: wear-service
          servicePort: 80
  - host: watch.my-online-store.com
    http:
      paths:
      - backend:
          serviceName: watch-service
          servicePort: 80

```

Lab 2 on Ingress Networking:
29  k get all --all-namespaces
30  clear
31  k create namespace ingress-space
32  k get namespaces
33  k create configmap nginx-configuration --namespace=ingress-space
34  k create serviceaccount ingress-serviceaccount --namespace=ingress-space

master $ k get roles,rolebindings --namespace=ingress-space
NAME                                          AGErole.rbac.authorization.k8s.io/ingress-role   69s

NAME                                                         AGE
rolebinding.rbac.authorization.k8s.io/ingress-role-binding   68s
