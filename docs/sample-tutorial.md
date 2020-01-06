# Sample Tutorial

[Exposing an External IP Address to Access an Application in a Cluster](https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/)

간단한 배포 튜토리얼

## 서비스 생성

```bash
kubectl apply -f example.yaml
```

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: load-balancer-example
  name: hello-world
spec:
  replicas: 3
  selector:
    matchLabels:
      app.kubernetes.io/name: load-balancer-example
  template:
    metadata:
      labels:
        app.kubernetes.io/name: load-balancer-example
    spec:
      containers:
      - image: gcr.io/google-samples/node-hello:1.0
        name: hello-world
        ports:
        - containerPort: 8080
```

### 확인

```bash
kubectl get pod

NAME                          READY   STATUS    RESTARTS   AGE
hello-world-f9b447754-755sk   1/1     Running   0          38s
hello-world-f9b447754-bqp64   1/1     Running   0          38s
hello-world-f9b447754-d7jnz   1/1     Running   0          37s
```

```bash
kubectl get deployments hello-world

NAME          READY   UP-TO-DATE   AVAILABLE   AGE
hello-world   3/3     3            3           45s
```

```bash
kubectl describe deployments hello-world

Name:                   hello-world
Namespace:              default
CreationTimestamp:      Sun, 05 Jan 2020 08:00:19 +0000
Labels:                 app.kubernetes.io/name=load-balancer-example
Annotations:            deployment.kubernetes.io/revision: 1
                        kubectl.kubernetes.io/last-applied-configuration:
                          {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app.kubernetes.io/name":"load-balancer-example"},"name..."
Selector:               app.kubernetes.io/name=load-balancer-example
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app.kubernetes.io/name=load-balancer-example
  Containers:
   hello-world:
    Image:        gcr.io/google-samples/node-hello:1.0
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   hello-world-f9b447754 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  69s   deployment-controller  Scaled up replica set hello-world-f9b447754 to 3
```

```bash
kubectl get replicasets

NAME                    DESIRED   CURRENT   READY   AGE
hello-world-f9b447754   3         3         3       95s
```

```bash
kubectl describe replicasets

Name:           hello-world-f9b447754
Namespace:      default
Selector:       app.kubernetes.io/name=load-balancer-example,pod-template-hash=f9b447754
Labels:         app.kubernetes.io/name=load-balancer-example
                pod-template-hash=f9b447754
Annotations:    deployment.kubernetes.io/desired-replicas: 3
                deployment.kubernetes.io/max-replicas: 7
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/hello-world
Replicas:       3 current / 3 desired
Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app.kubernetes.io/name=load-balancer-example
           pod-template-hash=f9b447754
  Containers:
   hello-world:
    Image:        gcr.io/google-samples/node-hello:1.0
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  104s  replicaset-controller  Created pod: hello-world-f9b447754-bqp64
  Normal  SuccessfulCreate  103s  replicaset-controller  Created pod: hello-world-f9b447754-755sk
  Normal  SuccessfulCreate  103s  replicaset-controller  Created pod: hello-world-f9b447754-kwrsd
```

```bash
kubectl expose deployment hello-world --type=LoadBalancer --name=my-service

service/my-service exposed
```

```bash
kubectl get services my-service

NAME         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
my-service   LoadBalancer   10.96.13.74   <pending>     8080:30779/TCP   9s
```

1. 만약 외부 IP 주소가 `pending`으로 표시되면 잠시 기다린 다음, 동일한 명령어를 다시 입력한다.
1. Vagrant 사용 시: [참고 링크](https://8gwifi.org/docs/kube-debug.jsp)


```bash
kubectl patch svc my-service  -p '{"spec": {"type": "LoadBalancer", "externalIPs":["192.168.91.101"]}}'

service/my-service patched
```

```bash
kubectl get services my-service

NAME         TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)          AGE
my-service   LoadBalancer   10.96.13.74   192.168.91.101   8080:30779/TCP   3m58s
```

```bash
# curl http://<external-ip>:<port>
curl http://192.168.91.101:8080
```

## 서비스 삭제

```bash
kubectl delete services my-service
kubectl delete deployment hello-world
```