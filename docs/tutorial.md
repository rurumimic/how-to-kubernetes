# Tutorial

[Exposing an External IP Address to Access an Application in a Cluster](https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/)

간단한 서비스 배포 튜토리얼

---

## Deployment

Deployment 스펙을 작성한다.

```bash
vi tutorial.yaml
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

### Deployment 생성

Deployment 객체와 ReplicaSet 객체를 생성한다.  
ReplicaSet은 3개의 Pod를 만든다.  
각 Pod는 Hello World 애플리케이션을 실행한다.

```bash
kubectl apply -f tutorial.yaml

# deployment.apps/hello-world created
```

### Deployment 정보 확인

Pod를 확인하면 `ContainerCreating`에서 `Running` 상태가 변경된다.

```bash
kubectl get pod

# NAME                          READY   STATUS    RESTARTS   AGE
# hello-world-f9b447754-755sk   1/1     Running   0          38s
# hello-world-f9b447754-bqp64   1/1     Running   0          38s
# hello-world-f9b447754-d7jnz   1/1     Running   0          37s
```

배포 정보를 확인한다.

```bash
kubectl get deployments hello-world

# NAME          READY   UP-TO-DATE   AVAILABLE   AGE
# hello-world   3/3     3            3           45s
```

자세한 배포 정보를 확인한다.

```bash
kubectl describe deployments hello-world

# Name:                   hello-world
# Namespace:              default
# CreationTimestamp:      Sun, 05 Jan 2020 08:00:19 +0000
# Labels:                 app.kubernetes.io/name=load-balancer-example
# Annotations:            deployment.kubernetes.io/revision: 1
#                         kubectl.kubernetes.io/last-applied-configuration:
#                           {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app.kubernetes.io/name":"load-balancer-example"},"name..."
# Selector:               app.kubernetes.io/name=load-balancer-example
# Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
# StrategyType:           RollingUpdate
# MinReadySeconds:        0
# RollingUpdateStrategy:  25% max unavailable, 25% max surge
# Pod Template:
#   Labels:  app.kubernetes.io/name=load-balancer-example
#   Containers:
#    hello-world:
#     Image:        gcr.io/google-samples/node-hello:1.0
#     Port:         8080/TCP
#     Host Port:    0/TCP
#     Environment:  <none>
#     Mounts:       <none>
#   Volumes:        <none>
# Conditions:
#   Type           Status  Reason
#   ----           ------  ------
#   Available      True    MinimumReplicasAvailable
#   Progressing    True    NewReplicaSetAvailable
# OldReplicaSets:  <none>
# NewReplicaSet:   hello-world-f9b447754 (3/3 replicas created)
# Events:
#   Type    Reason             Age   From                   Message
#   ----    ------             ----  ----                   -------
#   Normal  ScalingReplicaSet  69s   deployment-controller  Scaled up replica set hello-world-f9b447754 to 3
```

### ReplicaSet 정보 확인

ReplicaSet을 확인한다.

```bash
kubectl get replicasets

# NAME                    DESIRED   CURRENT   READY   AGE
# hello-world-f9b447754   3         3         3       95s
```

ReplicaSet 정보를 자세하게 확인한다.

```bash
kubectl describe replicasets

# Name:           hello-world-f9b447754
# Namespace:      default
# Selector:       app.kubernetes.io/name=load-balancer-example,pod-template-hash=f9b447754
# Labels:         app.kubernetes.io/name=load-balancer-example
#                 pod-template-hash=f9b447754
# Annotations:    deployment.kubernetes.io/desired-replicas: 3
#                 deployment.kubernetes.io/max-replicas: 7
#                 deployment.kubernetes.io/revision: 1
# Controlled By:  Deployment/hello-world
# Replicas:       3 current / 3 desired
# Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed
# Pod Template:
#   Labels:  app.kubernetes.io/name=load-balancer-example
#            pod-template-hash=f9b447754
#   Containers:
#    hello-world:
#     Image:        gcr.io/google-samples/node-hello:1.0
#     Port:         8080/TCP
#     Host Port:    0/TCP
#     Environment:  <none>
#     Mounts:       <none>
#   Volumes:        <none>
# Events:
#   Type    Reason            Age   From                   Message
#   ----    ------            ----  ----                   -------
#   Normal  SuccessfulCreate  104s  replicaset-controller  Created pod: hello-world-f9b447754-bqp64
#   Normal  SuccessfulCreate  103s  replicaset-controller  Created pod: hello-world-f9b447754-755sk
#   Normal  SuccessfulCreate  103s  replicaset-controller  Created pod: hello-world-f9b447754-kwrsd
```

---

## Service 생성

Deployment를 외부로 노출하여 Service 객체를 생성한다.

```bash
kubectl expose deployment hello-world --type=LoadBalancer --name=my-service

# service/my-service exposed
```

### Service 정보 확인

Service 정보를 확인한다.

```bash
kubectl get services my-service

# NAME         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
# my-service   LoadBalancer   10.96.13.74   <pending>     8080:30779/TCP   9s
```

만약 외부 IP 주소가 `pending`으로 표시되면 잠시 기다린 다음, 동일한 명령어를 다시 입력한다.

#### EXTERNAL-IP 상태가 계속 pending일 때

Load Balancer 서비스를 지원하지 않는 환경일 경우, `EXTERNAL-IP`가 `pending` 상태로 머무르며 `NodePort`처럼 행동한다.  

직접 Load Balancer IP를 할당해 문제를 해결한다.

```bash
kubectl patch svc my-service  -p '{"spec": {"type": "LoadBalancer", "externalIPs":["192.168.10.101"]}}'

# service/my-service patched
```

다시 Service 정보를 확인한다.

```bash
kubectl get services my-service

# NAME         TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)          AGE
# my-service   LoadBalancer   10.96.16.17   192.168.10.101   8080:32631/TCP   5m27s
```

## 서버 접속

이제 외부에서 `http://<external-ip>:<port>` 형식으로 접근할 수 있다.  
로컬 브라우저에서 서비스가 노출된 주소로 접속한다.

```bash
curl http://192.168.10.101:8080
```

**Hello Kubernetes!**

---

## 종료

서비스를 삭제하고 배포를 제거한다.

```bash
kubectl delete services my-service

# service "my-service" deleted
```

```bash
kubectl delete deployment hello-world

# deployment.apps "hello-world" deleted
```