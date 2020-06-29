# 선언형 관리

[구성 파일을 이용한 쿠버네티스 오브젝트의 선언형 관리](https://kubernetes.io/ko/docs/tasks/manage-kubernetes-objects/declarative-config/)

## 트레이드 오프

- 명령형 커맨드
- 명령형 오브젝트 구성
- 선언형 오브젝트 구성

## 명령어

```bash
kubectl diff -f https://k8s.io/examples/application/simple_deployment.yaml
kubectl apply -f https://k8s.io/examples/application/simple_deployment.yaml
kubectl get -f https://k8s.io/examples/application/simple_deployment.yaml -o yaml
```

업데이트:

```bash
kubectl diff -f https://k8s.io/examples/application/update_deployment.yaml
kubectl apply -f https://k8s.io/examples/application/update_deployment.yaml
kubectl get -f https://k8s.io/examples/application/update_deployment.yaml -o yaml
```

삭제:

```bash
kubectl delete -f https://k8s.io/examples/application/update_deployment.yaml
```

## 예제 오브젝트

처음:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

버전 수정:

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.16.1 # update the image
        ports:
        - containerPort: 80
```