## Pod Definition from Labs:

### Secret Definition:
```
cloud_user@ip-10-0-1-101:~$ cat candy-service-secret.yml
apiVersion: v1
kind: Secret
metadata:
  name: db-passwordstringData:
  password: Kub3rn3t3sRul3s!
```

### ConfigMap Definition:
candy-service-config.yml:
```
apiVersion: v1kind: ConfigMap
metadata:
  name: candy-service-config
data:
  candy.cfg: |-    candy.peppermint.power=100000000
    candy.nougat-armor.strength=10
cloud_user@ip-10-0-1-101:~$
```

### POD DEFINITION
candy-service-pod.yml:
```
apiVersion: v1kind: Pod
metadata:
  name: candy-service
spec:
  serviceAccountName: candy-svc  securityContext:
    fsGroup: 2000
  containers:
  - name: candy-service
    image: linuxacademycontent/candy-service:1
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-password
          key: password
    volumeMounts:
    - name: config-volume
      mountPath: /etc/candy-service
  volumes:
  - name: config-volume
    configMap:
      name: candy-service-config
```



kubectl describe pod candy-service
```
cloud_user@ip-10-0-1-101:~$ kd pod candy-service
Name:               candy-service
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               ip-10-0-1-103/10.0.1.103
Start Time:         Sat, 30 May 2020 00:43:19 +0000
Labels:             <none>
Annotations:        kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"candy-service","namespace":"default"},"spec":{"containers":[{"env":[{...
Status:             Running
IP:                 10.244.2.2
Containers:
  candy-service:    Container ID:   docker://1b70f12e51125ad444cb816f89d624dced40abd0dc2e734237f5a7a8f1e69e21
    Image:          linuxacademycontent/candy-service:1
    Image ID:       docker-pullable://linuxacademycontent/candy-service@sha256:eda64284ca388e265e54010d3000380e15d7c4df67e38ca3ec44d81f8402fa4e
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 30 May 2020 00:43:35 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  128Mi
    Requests:
      cpu:     250m
      memory:  64Mi
    Environment:
      DB_PASSWORD:  <set to the key 'password' in secret 'db-password'>  Optional: false
    Mounts:
      /etc/candy-service from config-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from candy-svc-token-tzssq (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      candy-service-config
    Optional:  false
  candy-svc-token-tzssq:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  candy-svc-token-tzssq
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From                    Message
  ----    ------     ----  ----                    -------
  Normal  Scheduled  78s   default-scheduler       Successfully assigned default/candy-service to ip-10-0-1-103
  Normal  Pulling    70s   kubelet, ip-10-0-1-103  pulling image "linuxacademycontent/candy-service:1"
  Normal  Pulled     62s   kubelet, ip-10-0-1-103  Successfully pulled image "linuxacademycontent/candy-service:1"
  Normal  Created    62s   kubelet, ip-10-0-1-103  Created container
  Normal  Started    62s   kubelet, ip-10-0-1-103  Started container
```
