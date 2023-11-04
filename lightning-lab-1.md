**1. Create a Persistent Volume called log-volume. It should make use of a storage class name manual. It should use RWX as the access mode and have a size of 1Gi. The volume should use the hostPath /opt/volume/nginx.**  
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: log-volume
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/opt/volume/nginx"
  storageClassName: manual
```

**Next, create a PVC called log-claim requesting a minimum of 200Mi of storage. This PVC should bind to log-volume.**  
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: log-claim
spec:
  resources:
   requests:
    storage: 200Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  storageClassName: manual
```

**Mount this in a pod called logger at the location /var/www/nginx. This pod should use the image nginx:alpine.**  
```
apiVersion: v1
kind: Pod
metadata:
  name: logger
spec:
  containers:
  - image: nginx:alpine
    name: logger
    volumeMounts:
    - name: log
      mountPath: /var/www/nginx
  volumes:
  - name: log
    persistentVolumeClaim:
     claimName: log-claim
```
**2. We have deployed a new pod called secure-pod and a service called secure-service. Incoming or Outgoing connections to this pod are not working.
Troubleshoot why this is happening.**  

**Make sure that incoming connection from the pod webapp-color are successful.**  
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: netpool
spec:
  podSelector:
    matchLabels:
     run : secure-pod
  policyTypes:
  - Ingress 
  ingress:
  - from:
     - podSelector:
        matchLabels:
         name: webapp-color
    ports: 
    - protocol: TCP
      port: 80
```

**3. Create a pod called time-check in the dvl1987 namespace. This pod should run a container called time-check that uses the busybox image.**
```
kubectl create ns dvl1987
```
**Create a config map called time-config with the data TIME_FREQ=10 in the same namespace.**  
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: time-config
  namespace: dvl1987
data:
  TIME_FREQ: "10"
```

**The time-check container should run the command: while true; do date; sleep $TIME_FREQ;done and write the result to the location /opt/time/time-check.log.**  
**The path /opt/time on the pod should mount a volume that lasts the lifetime of this pod.**  
```
apiVersion: v1
kind: Pod
metadata:
  name: time-check
  namespace: dvl1987
spec:
  containers:
  - image: busybox
    name: time-check
    env:
    - name: TIME_FREQ
      valueFrom:
        configMapKeyRef:
         name: time-config
         key: TIME_FREQ
    command: ["/bin/sh", "-c", "while true; do date; sleep $TIME_FREQ;done > /opt/time/time-check.log" ]
    volumeMounts:
    - name: volume-name
      mountPath: /opt/time
  volumes:
  - name: volume-name
    emptyDir: {}
 ```
    
**4. Create a new deployment called nginx-deploy, with one single container called nginx, image nginx:1.16 and 4 replicas. The deployment should use RollingUpdate strategy with maxSurge=1, and maxUnavailable=2.**    
**Next upgrade the deployment to version 1.17 using rolling update.**  
**Finally, once all pods are updated, undo the update and go back to the previous version.** 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy
  name: nginx-deploy
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx-deploy
  strategy: 
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 2
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx:1.16
        name: nginx
```
```
kubectl set image deploy/nginx-deploy nginx=nginx:1.17 --record=true
```
```
kubectl rollout undo deploy/nginx-deploy
```

**5. Create a redis deployment with the following parameters:**  
**Name of the deployment should be redis using the redis:alpine image. It should have exactly 1 replica.** 
**The container should request for .2 CPU. It should use the label app=redis.**  
**It should mount exactly 2 volumes:**  
**Make sure that the pod is scheduled on master/controlplane node.**  

**a. An Empty directory volume called data at path /redis-master-data.**  
**b. A configmap volume called redis-config at path /redis-master.**  
**c.The container should expose the port 6379.**  
```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: redis
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
 
  template:
    metadata:
      labels:
        app: redis
    spec:
      nodeName: controlplane
      containers:
      - image: redis:alpine
        name: redis-c
        ports:
        - containerPort: 6379
        resources:
          requests:
            cpu: "0.2"
        volumeMounts:
        - name: data
          mountPath: /redis-master-data
        - name: redis-config
          mountPath: /redis-master
      volumes:
      - name: data
        emptyDir: {} 
      - name: redis-config
        configMap:
          name: redis-config
```      
         
