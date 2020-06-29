# [단일 인스턴스 스테이트풀 애플리케이션 실행하기](https://kubernetes.io/ko/docs/tasks/run-application/run-single-instance-stateful-application/)

애플리케이션을 스케일링하지 않는다. 이 설정은 단일 인스턴스 애플리케이션 전용이다.  
기본적인 퍼시스턴트볼륨은 하나의 파드에서만 마운트할 수 있다.  
클러스터 형태의 스테이트풀 애플리케이션에 대해서는 [스테이트풀셋](https://kubernetes.io/ko/docs/concepts/workloads/controllers/statefulset/)을 보자.  
`strategy: type: Recreate`: 롤링 업데이트 사용 안 함.

## Run

```bash
kubectl apply -f https://k8s.io/examples/application/mysql/mysql-pv.yaml;
kubectl apply -f https://k8s.io/examples/application/mysql/mysql-deployment.yaml;
```

```bash
kubectl get pods;
kubectl get pv;
kubectl get pvc;
```

```bash
kubectl describe deployment mysql;
```

```bash
kubectl get pods -l app=mysql;
```

```bash
kubectl describe pvc mysql-pv-claim;
```

```bash
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -ppassword;

mysql> exit;
```

```bash
kubectl delete deployment,svc mysql;
kubectl delete pvc mysql-pv-claim;
kubectl delete pv mysql-pv-volume;
```

## Files

```yml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
  clusterIP: None
---
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
          # Use secret in real usage
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```

```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```