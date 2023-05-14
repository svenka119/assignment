## Create a Kubernetes In Docker(Kind) cluster, and start docker desktop
kind create cluster --config kind-config.yaml - creates a cluster
kubectl cluster-info --context kind-kind - verify the nodes

## Install nginx ingress in the controller

<details>
<summary> Ingress (Click to Expand) </summary>

```

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.1/deploy/static/provider/kind/deploy.yaml

Or helm
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
kubectl create namespace ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx

```
</details>

Fixed a few port changes in the ingress yaml and service yaml to route the traffic to the backend pods

## Install the helm chart in the cluster and a namespace test

<details>
<summary> Install the chart (Click to Expand)</summary>

```
helm install assignment-test . -n test    
NAME: assignment-test
LAST DEPLOYED: Sun May 14 16:28:12 2023
NAMESPACE: test
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

</details>

## To test traffic to the ingress controller from my local machine

I used a sudo command here as in my system there were permission issues

<details>
<summary> Port Forwarding (Click to Expand)</summary>

```

sudo kubectl port-forward svc/ingress-nginx-controller 80:80 -n ingress-nginx
Password:
Forwarding from 127.0.0.1:80 -> 80
Forwarding from [::1]:80 -> 80
Handling connection for 80
```
</details>

## Make a curl command to localhost:80/pfpt/test200  

<details>
<summary> Curl Command (Click to Expand)</summary>

```
curl  http://localhost:80/pfpt/test200  
Hello World%    

 curl  http://localhost:80/pfpt/test200  -v 
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 80 (#0)
> GET /pfpt/test200 HTTP/1.1
> Host: localhost
> User-Agent: curl/7.64.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Sun, 14 May 2023 11:04:04 GMT
< Content-Type: text/plain; charset=utf-8
< Content-Length: 11
< Connection: keep-alive
< 
* Connection #0 to host localhost left intact
Hello World* Closing connection 0         

```
</details>

## Logs:
<details>
<summary> Pods (Click to Expand)</summary>

```
kubectl describe pod -n test           
Name:         simple-dump-server-5684c8b8bf-tnjnk
Namespace:    test
Priority:     0
Node:         kind-worker/172.18.0.2
Start Time:   Sun, 14 May 2023 16:35:01 +0530
Labels:       app=simple-dump-server
              chart=assignment-test
              heritage=Helm
              pod-template-hash=5684c8b8bf
              release=assignment-test
Annotations:  <none>
Status:       Running
IP:           10.244.1.5
IPs:
  IP:           10.244.1.5
Controlled By:  ReplicaSet/simple-dump-server-5684c8b8bf
Containers:
  simple-dump-server:
    Container ID:   containerd://4bec473fb3a455ce89a06c7a6f220998d65acd40c08ac2e3f139edcd4cb061b3
    Image:          kousik93/simple-dump-server:assignment-test
    Image ID:       docker.io/kousik93/simple-dump-server@sha256:3dad43fc6ef6e56aeb84679c9a060fdeec8dadb2e9ff443824dba234c0133de5
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 14 May 2023 16:35:02 +0530
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     300m
      memory:  150Mi
    Requests:
      cpu:        200m
      memory:     100Mi
    Liveness:     http-get http://:8080/healthz delay=15s timeout=20s period=30s #success=1 #failure=3
    Readiness:    http-get http://:8080/readyz delay=15s timeout=20s period=30s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-nxqdw (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-nxqdw:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  38s   default-scheduler  Successfully assigned test/simple-dump-server-5684c8b8bf-tnjnk to kind-worker
  Normal  Pulled     38s   kubelet            Container image "kousik93/simple-dump-server:assignment-test" already present on machine
  Normal  Created    38s   kubelet            Created container simple-dump-server
  Normal  Started    38s   kubelet            Started container simple-dump-server

```
</details>

<details>
<summary>Services (Click to Expand)</summary>

```

kubectl describe svc -n test
Name:              simple-dump-server
Namespace:         test
Labels:            app=simple-dump-server
                   app.kubernetes.io/managed-by=Helm
                   chart=assignment-test
                   heritage=Helm
                   release=assignment-test
Annotations:       meta.helm.sh/release-name: assignment-test
                   meta.helm.sh/release-namespace: test
                   prometheus.io/scrape: true
Selector:          app=simple-dump-server
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.72.44
IPs:               10.96.72.44
Port:              http  8080/TCP
TargetPort:        8080/TCP
Endpoints:         10.244.1.5:8080
Session Affinity:  None
Events:            <none>
```
</details>

<details>
<summary>Ingress Resource (Click to Expand)</summary>

```

kubectl describe ingress -n test
Name:             simple-dump-server
Namespace:        test
Address:          
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /pfpt/test200   simple-dump-server:8080 (10.244.1.5:8080)
Annotations:  ingress.kubernetes.io/ingress.class: nginx
              kubernetes.io/ingress.class: nginx
              meta.helm.sh/release-name: assignment-test
              meta.helm.sh/release-namespace: test
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  Sync    115s  nginx-ingress-controller  Scheduled for sync

```
</details>

<details>
<summary>Deployments (Click to Expand) </summary>

```
kubectl describe deployment -n test
Name:                   simple-dump-server
Namespace:              test
CreationTimestamp:      Sun, 14 May 2023 16:35:01 +0530
Labels:                 app=simple-dump-server
                        app.kubernetes.io/managed-by=Helm
                        chart=assignment-test
                        heritage=Helm
                        release=assignment-test
Annotations:            deployment.kubernetes.io/revision: 1
                        meta.helm.sh/release-name: assignment-test
                        meta.helm.sh/release-namespace: test
                        service-version: 
Selector:               app=simple-dump-server
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:           app=simple-dump-server
                    chart=assignment-test
                    heritage=Helm
                    release=assignment-test
  Service Account:  simple-dump-server
  Containers:
   simple-dump-server:
    Image:      kousik93/simple-dump-server:assignment-test
    Port:       8080/TCP
    Host Port:  0/TCP
    Limits:
      cpu:     300m
      memory:  150Mi
    Requests:
      cpu:        200m
      memory:     100Mi
    Liveness:     http-get http://:8080/healthz delay=15s timeout=20s period=30s #success=1 #failure=3
    Readiness:    http-get http://:8080/readyz delay=15s timeout=20s period=30s #success=1 #failure=3
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   simple-dump-server-5684c8b8bf (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  2m19s  deployment-controller  Scaled up replica set simple-dump-server-5684c8b8bf to 1

```

</details>

<details>
<summary>Ingress Controller (Click to Expand)</summary>

```
kubectl get all -n ingress-nginx   
NAME                                           READY   STATUS    RESTARTS      AGE
pod/ingress-nginx-controller-697844d87-rpdwh   1/1     Running   3 (10m ago)   3d2h

NAME                                         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                     AGE
service/ingress-nginx                        NodePort       10.96.128.115   <none>        80:31720/TCP,443:31782/TCP,8443:32723/TCP   3d
service/ingress-nginx-controller             LoadBalancer   10.96.87.148    <pending>     80:32497/TCP,443:31926/TCP                  3d2h
service/ingress-nginx-controller-admission   ClusterIP      10.96.173.30    <none>        443/TCP                                     3d2h

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           3d2h

NAME                                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-697844d87   1         1         1       3d2h
                     
```
</details>

<details>
<summary>All Resource in Test (Click to Expand)</summary>

```
 kubectl get all -n test         
NAME                                      READY   STATUS    RESTARTS   AGE
pod/simple-dump-server-5684c8b8bf-tnjnk   1/1     Running   0          4m26s

NAME                         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/simple-dump-server   ClusterIP   10.96.72.44   <none>        8080/TCP   4m26s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/simple-dump-server   1/1     1            1           4m26s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/simple-dump-server-5684c8b8bf   1         1         1       4m26s

```
</details>

