CKAD Notes

# H1 References:
https://nbtechsolutions.wordpress.com/2020/04/30/ckad-experience/
https://medium.com/@ikaboubi/my-feedback-about-cka-and-ckad-e82a35585fe9
https://codeburst.io/kubernetes-ckad-weekly-challenges-overview-and-tips-7282b36a2681
https://techbeacon.com/enterprise-it/47-advanced-tutorials-mastering-kubernetes

Introduction:
CNCF Certification References:
Certified Kubernetes Application Developer: https://www.cncf.io/certification/ckad/

Candidate Handbook: https://www.cncf.io/certification/candidate-handbook

Exam Tips: https://www2.thelinuxfoundation.org/ckad-tips
Course Deck

How K8 Works.

Exam - 2 hours
Performance Knowledge.
Kubernetes Documentation.

Keep the code - DEVOPS15 - handy while registering for the CKA or CKAD exams at Linux Foundation to get a 15% discount.

#H2 Tips:
1> k8.io Documentation
2> Understand Concepts in Exam Curriculum (Link the Curriculum)
3> Be Good at YAML - Take course in LA
4> Time Management:
   > nano editor - Document and Practice ShortCuts
   > kubectl alias
   > Set Context and Namespaces (kubectl config set-context <cluster-name> --namespace=myns)
   > Explain (kubectl explain cronjob.spec.jobTenplate --recursive)
   > Know your Kubectl Commands - https:/ /kubernetes.io/docs/reference/generated/kubectl/kubectl-commands
   > --restart (YAML generators)
   		> kubectl run nginx --image=nginx (deployment)
   		> kubectl run nginx --image=nginx --restart=Never (pod)
   		> kubectl run nginx --image=nginx --restart=OnFailure (job)
   		> kubectl run nginx --image=nginx --restart=OnFailure --schedule="****" (cronJob)

   > --dry-run command
   > UNIX Bash
   	> https://www.youtube.com/watch?v=rnemKrveZks&feature=youtu.be
   	  > 21 mts TIme Line
   > Grep
   > GO thru all Questions ?

=============================================================
 Section 2: Core Concepts:
 Kubernetes Architecture
 Installing Kube gives you the following:
  - API Server
  - etcd
  - Kubelet - Agent on Worker Nodes
  - Container RunTime
  - Controller - main Orchestrater
  - Scheduler

==============================================================
Where does the master live in a multi Node Cluster Architecture. ?
What if Node containing master goes down.
Container Engine:
 - Docker
 - rkt
 - CRI-O

kubectl run hello-minikube # deploy a configuration or cluster
kubctl cluster-info
kubectl get nodes


Pods:
==========================================================
> Containers are encapsulated into Pod(Kubernetes Object)
> Pod :: smallest object in Kubernetes

Pod can have one or more containers of a different kind.
Helper Container in the same pod ?

Concept of Corrpution of Shared VOlume or rogue Pod ?

How To Deploy Pod:
kubctl run nginx --image nginx
kubectl get pods
--------------------------------------------------------
YAML :
Pod Defnition
kind:
 Pod
 Service
 ReplicaSet
 Deployment

----
apiVersion:
kind:
metadata:

spec:
----
Child label should be to right of parent.
metadata:
  name: myapp-pod
  labels:
    app: myapp
    type: front-end

spec: //is a dictionary
  containers:  // Is a List or Array
    - name: nginx-container //Dictionary
      image: nginx

 =========================
 Deployments:

 kubectl run nginx --image=nginx --restart=Never --dry-run -o yaml
 kubectl create deployment --image=nginx nginx --dry-run -o yaml

 kubectl create deployment does not have a --replicas option. You could first create it and then scale it using the kubectl scale command

 Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)::
 kubectl create deployment --image=nginx nginx --dry-run -o yaml > nginx-deployment.yaml
 You can then update the YAML file with the replicas or any other field before creating the deployment


 Service:

 CLUSTERIP:
 Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379
 kubectl expose pod redis --port=6379 --name redis-service --dry-run -o yaml
 // This will automatically use the pod's labels as selectors
 OR
 B>
 kubectl create service clusterip redis --tcp=6379:6379 --dry-run -o yaml
 //(This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)

 ===========
 NODEPORT
 Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

 kubectl expose pod nginx --port=80 --name nginx-service --dry-run -o yaml

 (This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

  Or

 kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run -o yaml

 (This will not use the pods labels as selectors)

 Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.


====================
FORMATTING:

kubectl [command] [TYPE] [NAME] -o <output_format>

Here are some of the commonly used formats:

-o jsonOutput a JSON formatted API object.

-o namePrint only the resource name and nothing else.

-o wideOutput in the plain-text format with any additional information.

-o yamlOutput a YAML formatted API object.

=======================

Namespace:

default
kube-system
kube-public
> for prod purposes create your own namespace.
- dev
- prod
Each namespace has it own Policy ?
Namespace Resource Limit.

Each Service has a DNS Entry

Example of DNS:
mysql.connect("db-service.dev.svc.cluster.local")
db-service    :: Service Name
dev           :: NameSpace
svc           :: Service
cluster.local :: domain
------
kubectl get pods
kubectl get pods --namepsace=kube-system

you can move namespace tag into pod-definition.yml file to ensure resource would get created always in the same namespace.

Create NameSpace Options:
namespace-dev.yml

apiVersion: v1
kind: Namespace
metadata:
  name: dev

> kubectl create -f namespace-dev.yml

OR
> kubectl create namespace dev

Change to a Namespace permanently:
kubectl config set-context $(kubectl config current-context) --namespace=dev
> kubectl get pods // for dev

# List Pods across all namespaces:
kubectl get pods --all-namespaces


contexts are used to manage clusters

How To Limit Resources in a NameSpace:
Use ResourceQuota

compute-quota.yaml
----------------------
apiVersion: v1
kind: ResourceQuota
metadata:
	name: compute-quota
	namespace: dev
spec:
	hard:
	    pods: "10"
	    requests.cpu: "4"
	    requests.memory: 5Gi
	    limits.cpu: "10"
	    limits.memory: 10Gi
--------------------------
===========

CONFIGURATION:

Commands and Arguments in Docker
docker run ubuntu
docker ps => the container is exited.
Container Exists if the Process within Docker stops.

CMD vs ENTRYPOINT

---
FROM Ubuntu
CMD  sleep 5.  #container will goto sleep
---
FROM Ubuntu
ENTRYPOINT ["sleep"]
-------
for above entrypoint:
docker run ubuntu-sleeper 10
--

FROM Ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]

> docker run ubuntu-sleeper
> docker run ubuntu-sleeper 10

Change Entrypoint:
docker run --entrypoint sleep2.0 ubuntu-sleeper 10

Kubernetes - Commands and Arguments:

pod-definiton.yml
```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
  image: ubuntu-sleeper
  command: ["sleep2.0"].  <=== ENTRYPOINT
  arg: ["10"]             <=== CMD
```
kubectl create -f pod-definition.yml

======
Editing Pods and Deployment:

kubectl edit pod <pod name>
kubectl delete pod webapp ==> this will trigger a new pod to be spun up
---
vi my-new-pod.yaml

Then delete the existing pod

kubectl delete pod webapp

Then create a new pod with the edited file

kubectl create -f my-new-pod.yaml
---


kubectl edit deployment <deployment name>
=====================

ConfigMaps:


Imperative:
----------
kubectl create configmap
   <config-name> --from-literal=<key>=<value>

kubectl create configmap
   app-config --from-literal=APP_COLOR=blue \
 			  --from-literal=APP_MOD=prod

kubectl create configmap
   <config-name> --from-file=<path-to-file>

----------------
Declarative:
--------

Config Map Definition

config-map.yaml
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```
kubectl create -f config-map.yaml
----------------

kubectl get configmaps
kubectl describe configmaps
-------------------------------

Pod Injection for a ConfigMap:
config-map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod


------------------
pod-definiton.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    envFrom:
      - configMapRef:
          name: app-config
```
kubectl create -f pod-definition.yaml

ConfigMap in Pods:
3 ways:

envFrom:
  - configMapRef:
      name: app-config

env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR


volumes:
- name: app-config-volume
  configMap:
    name: app-config

--------------------------

Secrets:

Secret:
DB_Host:  mysql
DB_User:  root
DB_Password:  paswrd

Secrets are stored in an Encrypted/Encoded Manner

Imperative:
> kubectl create secret generic
       <secret-name> --from-literal=<key>=<value> \
       <secret-name> --from-literal=<key>=<value>

> kubectl create secret generic
       <secret-name> --from-file=<path-to-file>

Declarative:
> kubectl create -f <file-name>

secret-data.yml
---------------
aoiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host:  mysql
  DB_User:  root
  DB_Password:  paswrd

THE ABOVE DATA SHOULD BE IN ENCODED FORMAT
How to convert data to encoded : use base64 conversion

echo -n 'mysql' | base64 <=== Encoding


----
> kubectl get secrets
> kubectl describe secrets
> kubectl get secret app-secret -o yaml

How to decode secret values:
echo -n 'bXlzcWw=' | base64 <== Decode

------
Inject ENV Variable into Pod , Secret here:

pod-definiton.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    envFrom:
      - secretRef:
          name: app-config
```
Note: If Creating Secrets in Pods as Volume
Each File is Created for a Secret
cat /opt/app-secret-volumes/DB_Passwrd

FollowUp:
- Do not check in secret object definition files to source code
- Enabling Encryption at Rest for Secrets so they are stored Encrypted in ETCD

------
Also the way kubernetes handles secrets. Such as:

- A secret is only sent to a node if a pod on that node requires it.
- Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
- Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.
------------
Practice Test:
```
kubectl create secret generic \
       db-secret --from-literal=DB_Host=sql01 \
       --from-literal=DB_User=root --from-literal=DB_Password=password123
```
======

Docker Security:

Docker runs on its own namespace while on a Host.
Namespace also leads to process isolation.
Docker runs processes with limited number of capabilities.
example to enable privilege:
docker run --cap-add MAC_ADMIN ubuntu
docker run --previliged ubuntu

===================================
Security Context:

Pod Definition: where Security Context is in scope of Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:
    runAsUser: 1000

  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
```
--------
Pod Definition: where Security Context is in scope of Container
```
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 1000
        capabilities:
          add: ["MAC_ADMIN"]
```
Note: Capabilties are only supported at the container level and not at the Pod Level
----------------------
Service Accounts:
================

Two Types of Services in K8:
USER Account    - By USERS
SERVICE Account - By MACHINES like Jenkins or Prometheus

> kubectl create serviceaccount dashboard-sa
> kubectl get serviceaccount
> kubectl describe serviceaccount dashboard-sa
  -- lists a Token which gets generated and used by an Application to authenticate with Kube API
> kubectl describe secret dashboard-sa-token-kbbdm

Service Account Token living within a Secret Object can be mounted as a volume within
a Kubernetes Cluster hence simpliyfing the process of exporting a token for a service
like Prometheus.


**Labs:**
Permissions added for newly created dashboard-sa account using RBAC:
master $ cat /var/rbac/dashboard-sa-role-binding.yaml
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: dashboard-sa # Name is case sensitive
  namespace: default
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io

------------------------------------
pod-reader-role.yaml

master $ cat /var/rbac/pod-reader-role.yaml
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups:
  - ''
  resources:
  - pods
  verbs:
  - get
  - watch
  - list

=====================================
Resource Requirements for Pods:
- CPU
- Memory
- Disk

pod-definiton.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
      limits:
        memory: "2Gi"
        cpu: 2
```
CPU => Throttling happens
Memory ==> Termination of Container
--------------------
LimitRange in that Namespace:
MEMORY
```
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

----------
CPU:
```
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 1
    defaultRequest:
      cpu: 0.5
    type: Container
```
-----------------------

Taints and Tolerations:
