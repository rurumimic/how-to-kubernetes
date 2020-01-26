# Horizontal Pod Autoscaler

- K8S Docs
  - [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
  - [Horizontal Pod Autoscaler Walkthrough](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/)
- EKS Workshop
  - [Implement autoscaling with HPA and CA](https://eksworkshop.com/beginner/080_scaling/)

## Before you begin

- a kubernetes cluster
- kubectl >= 1.10
- [metrics-server](https://github.com/kubernetes-sigs/metrics-server) monitoring

## Minikube

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

### addon metric-server

미니큐브 애드온 리스트를 확인한다. 비활성화된 `metrics-server`를 확인할 수 있다.

```bash
minikube addons list
# - metrics-server: disabled
```

이제 `metrics-server` 애드온을 활성화한다.

```bash
minikube addons enable metrics-server
# ✅ metrics-server was successfully enabled
```

미니큐브 애드온 리스트를 다시 확인한다.

```bash
minikube addons list
# - metrics-server: enabled
```

## Run & expose php-apache server

`k8s.gcr.io/hpa-example` 도커 이미지는 CPU가 부담을 받도록 만들어진 서버다.

```php
<?php
  $x = 0.0001;
  for ($i = 0; $i <= 1000000; $i++) {
    $x += sqrt($x);
  }
  echo "OK!";
?>
```

`php-apache` 서비스를 배포한다.

```bash
kubectl run php-apache \
--image=k8s.gcr.io/hpa-example \
--requests=cpu=200m \
--limits=cpu=500m \
--expose \
--port=80
```

`php-apache` deployment와 service가 생성됐다.

```bash
service/php-apache created
deployment.apps/php-apache created
```

## Create Horizontal Pod Autoscaler

autoscaler를 생성한다. 모든 Pod의 CPU 사용률이 평균 50%가 넘으면 최대 10개까지 복제를 늘린다.

```bash
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
# horizontalpodautoscaler.autoscaling/php-apache autoscale
```

autoscaler 상태를 확인한다.

```bash
kubectl get hpa
# NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
# php-apache   Deployment/php-apache   0%/50%    1         10        1          116s
```

## Increase load

새로운 터미널 창에서 다음 명령어를 실행한다.

```bash
kubectl run --generator=run-pod/v1 -it --rm load-generator --image=busybox /bin/sh
```

다음과 같은 안내가 출력된다.

```bash
If you don't see a command prompt, try pressing enter.
/ # 
```

`php-apache` 서비스에 접속하는 쉘 스크립트를 실행한다.

```bash
while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done
```

첫번째 터미널에서 autoscaler 상태를 확인한다.

```bash
kubectl get hpa
# NAME         REFERENCE               TARGETS    MINPODS   MAXPODS   REPLICAS   AGE
# php-apache   Deployment/php-apache   247%/50%   1         10        5          8m22s
```

deployment를 확인하면 복제 수가 늘어난 것을 확인할 수 있다.

```bash
kubectl get deployment php-apache
# NAME         READY   UP-TO-DATE   AVAILABLE   AGE
# php-apache   5/5     5            5           8m46s
```

## Stop load

두번째 터미널에서 `<Ctrl> + C`를 입력해 쉘 스크립트를 종료한다.

몇 분 뒤, 첫번째 터미널에서 autoscaler를 다시 확인하면 deployment가 1개로 줄어든 것을 볼 수 있다.

```bash
kubectl get hpa
# NAME         REFERENCE               TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
# php-apache   Deployment/php-apache   0%/50%    1         10        1          15m
```

```bash
kubectl get deployment php-apache
# NAME         READY   UP-TO-DATE   AVAILABLE   AGE
# php-apache   1/1     1            1           16m
```

## Delete service and deployment

```bash
kubectl delete deployment,service php-apache
```

## FAQ

- [metrics-server unable to authenticate to apiserver](https://github.com/kubernetes-sigs/metrics-server/issues/278)
