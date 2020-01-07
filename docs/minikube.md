# Minikube

[K8S: Installing Kubernetes with Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)

미니큐브로 로컬에서 쿠버네티스를 쉽게 사용해볼 수 있다.  
미니큐브는 노드가 하나인 쿠버네티스 클러스터를 VM에서 실행한다.  
쿠버네티스를 간단하게 연습하거나 개발용으로 사용하기 좋다.

## Minikube 기능

몇가지 쿠버네티스 기능을 사용할 수 있다.

- DNS
- NodePorts
- ConfigMaps, Secrets
- Dashboards
- Container Runtime: Docker, CRI-O, containerd
- Container Network Interface
- Ingress

---

## 설치

[미니큐브 설치](/docs/install-minikube.md)

---

## 실습

시작부터 사용, 삭제까지 실습해본다.

### 시작

미니큐브를 시작한다. (다양한 옵션: [K8S: Minikube. Starting a Cluster](https://kubernetes.io/docs/setup/learning-environment/minikube/#starting-a-cluster))

```bash
minikube start
```

```bash
😄  minikube v1.6.2 on Darwin 10.14.6
✨  Automatically selected the 'virtualbox' driver (alternates: [vmwarefusion])
🔥  Creating virtualbox VM (CPUs=2, Memory=2000MB, Disk=20000MB) ...
🐳  Preparing Kubernetes v1.17.0 on Docker '19.03.5' ...
🚜  Pulling images ...
🚀  Launching Kubernetes ... 
⌛  Waiting for cluster to come online ...
🏄  Done! kubectl is now configured to use "minikube"
```

### 클러스터 확인

[K8S: Interacting with Your Cluster](https://kubernetes.io/docs/setup/learning-environment/minikube/#interacting-with-your-cluster)

- [Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/): `minikube dashboard`

### 배포

간단한 HTTP 서버를 배포한다.

```bash
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10
# deployment.apps/hello-minikube created
```

### 서비스 노출

`NodePort` 타입으로 서비스를 노출한다.

```bash
kubectl expose deployment hello-minikube --type=NodePort --port=8080
# service/hello-minikube exposed
```

### Pod 확인

Pod는 실행되었지만 서비스가 노출되려면 잠시 기다려야 한다.

```bash
kubectl get pod
```

`STATUS`가 `ContainerCreating`에서 `Running`으로 바뀔 때까지 기다린다.

```
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-797f975945-ddrdk   1/1     Running   0          86s
```

### 서비스 접속

서비스로 접속할 수 있는 URL을 얻는다.

```bash
minikube service hello-minikube --url
# http://192.168.99.100:31831
```

브라우저로 접속해본다.

### 서비스 삭제

```bash
kubectl delete services hello-minikube
```

### 배포 삭제

```bash
kubectl delete deployment hello-minikube
```

### 클러스터 정지

```bash
minikube stop
```

### 클러스터 삭제

```bash
minikube delete
```