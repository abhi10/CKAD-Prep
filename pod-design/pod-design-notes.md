
#H3 High Level Overview:
==========
- Labels, Selectors and Annotations
- Rolling Updates and Rollbacks in Deployments
- Jobs and Cron Jobs

#H3 Labels, Selectors and Annotations
==========
- Labels::    Used to tag/identify an object, where object = Pod,Node etc.
- Selectors:: Used to filter objects based on their corresponding labels.
- Annotations :: Provide Extra Information.
Example Command:
> k get pods --selector tier=front-end


#H3 Rolling Updates and Rollbacks in Deployments
==========
Deployment Strategy:
- Recreate
- Rolling Update

> kubectl rollout deployment/myapp-deployment
> k describe deployment <deployment-name>
> kubectl get replicasets


Rollback:
> kubectl rollout undo deployment/myapp-deployment

**The following cmd created a Pod and Deployment as well:**
> kubectl run nginx --image=nginx

Command Summarization:
```
CREATE::   > kubectl create -f deployment-definition.yml
GET::      > kubectl get Deployments
UPDATE::   > kubectl apply -f deployment-definition.yaml
2>         > kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
STATUS::   > kubectl rollout status deployment/myapp-deployment
2>         > kubectl rollout history deployment/myapp-deployment
ROLLBACK:: > kuebctl rollback undo deployment/myapp-deployment
```

#H3 Updating a Deployment
====
Here are some handy examples related to updating a Kubernetes Deployment:
Creating a deployment, checking the rollout status and history:

In the example below, we will first create a simple deployment and inspect the rollout status and the rollout history:

```
master $ kubectl create deployment nginx --image=nginx:1.16
deployment.apps/nginx created

master $ kubectl rollout status deployment nginx
Waiting for deployment "nginx" rollout to finish: 0 of 1 updated replicas are available...
deployment "nginx" successfully rolled out


master $ kubectl rollout history deployment nginx
deployment.extensions/nginx
REVISION CHANGE-CAUSE
1     <none>

Using the --revision flag:
```
Here the revision 1 is the first version where the deployment was created.

You can check the status of each revision individually by using the --revision flag:
```
master $ kubectl rollout history deployment nginx --revision=1
deployment.extensions/nginx with revision #1

Pod Template:
 Labels:    app=nginx    pod-template-hash=6454457cdb
 Containers:  nginx:  Image:   nginx:1.16
  Port:    <none>
  Host Port: <none>
  Environment:    <none>
  Mounts:   <none>
 Volumes:   <none>
```
Using the --record flag:

You would have noticed that the "change-cause" field is empty in the rollout history output. We can use the --record flag to save the command used to create/update a deployment against the revision number.
```
master $ kubectl set image deployment nginx nginx=nginx:1.17 --record
deployment.extensions/nginx image updated
master $master $

master $ kubectl rollout history deployment nginx
deployment.extensions/nginx

REVISION CHANGE-CAUSE
1     <none>
2     kubectl set image deployment nginx nginx=nginx:1.17 --record=true
master $
```

You can now see that the change-cause is recorded for the revision 2 of this deployment.

Let's make some more changes. In the example below, we are editing the deployment and changing the image from nginx:1.17 to nginx:latest while making use of the --record flag.
```
master $ kubectl edit deployments. nginx --record
deployment.extensions/nginx edited

master $ kubectl rollout history deployment nginx
REVISION CHANGE-CAUSE
1     <none>
2     kubectl set image deployment nginx nginx=nginx:1.17 --record=true
3     kubectl edit deployments. nginx --record=true

master $ kubectl rollout history deployment nginx --revision=3
deployment.extensions/nginx with revision #3
```
Pod Template: Labels:    app=nginx
    pod-template-hash=df6487dc Annotations: kubernetes.io/change-cause: kubectl edit deployments. nginx --record=true

 Containers:
  nginx:
  Image:   nginx:latest
  Port:    <none>
  Host Port: <none>
  Environment:    <none>
  Mounts:   <none>
 Volumes:   <none>

**Undo a change**

Lets now rollback to the previous revision:
```
master $ kubectl rollout undo deployment nginx
deployment.extensions/nginx rolled back

master $ kubectl rollout history deployment nginx
deployment.extensions/nginxREVISION CHANGE-CAUSE
1     <none>
3     kubectl edit deployments. nginx --record=true
4     kubectl set image deployment nginx nginx=nginx:1.17 --record=true


master $ kubectl rollout history deployment nginx --revision=4
deployment.extensions/nginx with revision #4Pod Template:
 Labels:    app=nginx    pod-template-hash=b99b98f9
 Annotations: kubernetes.io/change-cause: kubectl set image deployment nginx nginx=nginx:1.17 --record=true
 Containers:
  nginx:
  Image:   nginx:1.17
  Port:    <none>
  Host Port: <none>
  Environment:    <none>
  Mounts:   <none>
 Volumes:   <none>


master $ kubectl describe deployments. nginx | grep -i image:
  Image:    nginx:1.17
```


- With this, we have rolled back to the previous version of the deployment with the image = nginx:1.17.

**Lab:**
webapp-service had multiple endpoints, why ?
```
master $ k describe service/webapp-service
Name:                     webapp-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 name=webapp
Type:                     NodePort
IP:                       10.98.102.116
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30080/TCP
Endpoints:                10.44.0.1:8080,10.44.0.2:8080,10.44.0.3:8080 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
master $
----------
master $ ./curl-test.sh
Hello, Application Version: v1 ; Color: blue OK

Hello, Application Version: v1 ; Color: blue OK
Upgrade application by setting image:
kubectl set image deployment frontend simple-webapp=kodekloud/webapp-color:v2 --record=true

Change the deployment strategy to 'Recreate'
>  'kubectl edit deployment frontend'
```
=====================
#H4 Jobs:
=======
Different Types of Workload:
Long Term/Short Term
- BatchProcessing
- Analytics
- Reporting
Example:
- Image Processing
- Email Reporting
-
How the above workloads work in Docker.

Kubernetes has a RestartPolicy by default
The container keeps restarting when the job Completes.
RestartPolicy => Change from Always to Never.

BatchProcessing:
- Compute Parallel Jobs.
- Kubernetes Job:
---------------------
job-definition.yaml
```
apiVersion: batch/v1
kind: Job
metadata:
	name: math-add-job
spec:
  completions: 3   # 3 jobs are created. Only Sequential
  parallelism: 3
	template:
	    spec:
        containers:
        - name: math-add
	        image: ubuntu
          command: ['expr', '3', '+', '2']

        restartPolicy: Never
```
----------------
> kubectl create -f job-definiiton.yaml
> kubectl get jobs
> kubect get pods # should not restart
> kubectl logs math-add-job-<suffix>
> kubectl delete job math-add-<suffix>
----------------

#H4 CronJobs:
=======
Jobs which are scheduled.
Example Definition:
cronjob-definition.yamlOutputjob-definition.yaml
```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
	name: reporting-cron-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
  spec:
    completions: 3   # 3 jobs are created. Only Sequential
    parallelism: 3   # adds parallelism effect
    template:
        spec:
          containers:
          - name: math-add
            image: ubuntu
            command: ['expr', '3', '+', '2']

```
kubectl create -f cron-job-definition.yaml
kubectl get cronjob
