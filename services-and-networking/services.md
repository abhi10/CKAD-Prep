# H1 Services in Kubernetes

Services enable communication within and outside the cluster.
Services Enable Loose Coupling between Microservices in our application.

# H3 Different Types of Services:
- NodePort
- ClusterIP
- LoadBalancer

# H3 NodePort
NodePort has its own IP Address
Port of Node is used to access the Webserver Itself
example port: 30000 - 32767
Service is Highly Flexible and Adaptive.

Service Definition:
```
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
  - targetPort: 80
    port: 80
    nodePort: 30008
  selectors:
    app: myapp
    type: front-end
```
> k create -f service-definition.yaml
> k get services

Access Web Server using:
> curl http://<webserver-ip-address>

What if there are Multiple Pods:
- Service automatically selects one of pods using selectors.
- Service acts like a builtin Load Balancer.

Multiple Nodes:
K8 maps Services across multiple Nodes across a Cluster.

# H3 Service - ClusterIP

EXAMPLE Cluster:
frontend Services
backend Services
Redis
Some Notes:
- All pods have IP addresses which are not Static as Pods go up and down.
- K8 Service would help in grouping pods together and help in access the group.
- Single Interface for backend
- Single Interface for Redis.
- Each Service has a name and IP Used by Pods to access the Service.
- Such a Service is termed ClusterIP
 SERVICE DEFINITION:
 ```
 apiVersion: v1
 kind: Service
 metadata:
   name: backend
 spec:
   type: ClusterIP
   ports:
   - targetPort: 80
     port: 80
   selectors:
     app: myapp
     type: back-end
 ```
 kubectl create -f service-definition.yaml
 kubectl get services
 ClusterIP/Service Name used by Pods for communication.
