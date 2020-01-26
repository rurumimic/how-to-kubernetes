# HA 클러스터 구축: external etcd

[K8S: External etcd topology](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/#external-etcd-topology)

![kubeadm HA topology - external etcd](https://d33wubrfki0l68.cloudfront.net/ad49fffce42d5a35ae0d0cc1186b97209d86b99c/5a6ae/images/kubeadm/kubeadm-ha-topology-external-etcd.svg)

---

## 구축 순서

1. Load Balancer
2. Etcd cluster
3. Control Plane Nodes
4. Worker Nodes
5. Deploy Service

## 환경 준비

- Ubuntu 16.04.6 LTS (Xenial Xerus)
- Control Plane x3:
  - RAM: 2GB
  - CPU: 2
- Load Balancer, Etcd x3, Worker x3:
  - RAM: 1GB
  - CPU: 1

---

## Vagrant 설정

자세한 Vagrant 사용 방법: [rurumimic/how-to-vagrant](https://github.com/rurumimic/how-to-vagrant)

[Vagrantfile](/vagrant/external/ubuntu/Vagrantfile)을 빈 폴더에 저장한다.

- Load Balancer x 1
  - 192.168.10.101
- Control Plane x 3: 2GB RAM, 2 CPU
  - 192.168.10.111 ~ 113
- Etcd x 3
  - 192.168.10.121 ~ 123
- Worker x 3
  - 192.168.10.131 ~ 133

---

## 로드밸런서

로드밸런서를 준비한다. HAProxy를 사용한다.

다음 과정을 따라 한다: **[HAProxy를 사용한 로드밸런서 구축](/docs/loadbalancer.md)**

---

## Etcd

다음 과정을 따라 한다: **[etcd cluster 구축](/docs/etcd-cluster.md)**

---

## Control Plane

다음 과정을 따라 한다: **[kubeadm 설치](/docs/install-kubeadm.md)**

---

## Etcd 연결

[K8S: Set up the etcd cluster](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#set-up-the-etcd-cluster)

`etcd-1` 호스트의 공개키를 `controlplane-1` 호스트에 저장한다.

```bash
root@etcd-1 $ cat /root/.ssh/id_rsa.pub
root@controlplane-1 $ vi /root/.ssh/authorized_keys
```

`etcd-1`에서 `controlplane-1`으로 SSH 접속을 확인한다.

```bash
root@etcd-1 $ ssh -A 192.168.10.111
# Are you sure you want to continue connecting (yes/no)? yes
root@controlplane-1:~$ exit
root@etcd-1 $
```

`controlplane-1`에서 디렉터리를 생성한다.

```bash
mkdir -p /etc/kubernetes/pki/etcd
```

etcd-1에서 controlplane-1으로 파일을 전송한다.

```bash
CONTROL_PLANE="root@192.168.10.111";
scp /etc/kubernetes/pki/etcd/ca.crt "${CONTROL_PLANE}":/etc/kubernetes/pki/etcd/ca.crt;
scp /etc/kubernetes/pki/apiserver-etcd-client.crt "${CONTROL_PLANE}":/etc/kubernetes/pki/apiserver-etcd-client.crt;
scp /etc/kubernetes/pki/apiserver-etcd-client.key "${CONTROL_PLANE}":/etc/kubernetes/pki/apiserver-etcd-client.key;
```

---

## 첫번째 Control Plane 실행

[K8S: Set up the first control plane node](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#set-up-the-first-control-plane-node)

root 계정이 아니어도 된다.  
`kubeadm-config.yaml` 파일을 작성한다.

- controlPlaneEndpoint: `LOAD_BALANCER_DNS`:`LOAD_BALANCER_PORT`
- endpoints: `ETCD_0_IP`, `ETCD_1_IP`, `ETCD_2_IP`

`vi ~/kubeadm-config.yaml`

```yml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: stable
controlPlaneEndpoint: "192.168.10.101:6443"
etcd:
    external:
        endpoints:
        - https://192.168.10.121:2379
        - https://192.168.10.122:2379
        - https://192.168.10.123:2379
        caFile: /etc/kubernetes/pki/etcd/ca.crt
        certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
        keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
```

첫번째 Control Plane의 kubeadm을 실행한다.

```bash
sudo kubeadm init --config kubeadm-config.yaml --upload-certs
```

다음과 같이 결과가 나타나면 성공이다.

```bash
Your Kubernetes control-plane has initialized successfully!
```

먼저 리눅스 일반 계정으로 쿠버네티스를 사용하기 위해서 권한을 설정한다.

```bash
mkdir -p $HOME/.kube;
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config;
sudo chown $(id -u):$(id -g) $HOME/.kube/config;
```

CNI 플러그인을 설치한다.

간단하게 Weave Net을 설정한다.

```bash
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

---

## Control Plane 클러스터 연결

`controlplane-2`, `controlplane-3` 노드에서 다음 명령어를 실행한다.

```bash
sudo kubeadm join 192.168.10.101:6443 --token jjnixg.bbg4e7amhd32vi1r \
    --discovery-token-ca-cert-hash sha256:fb36418e348299619133891bc1ae1dafee6facf8ef8962c135d6ba7b357f00e1 \
    --control-plane --certificate-key 06c4a3bb5a5525014c5c358991de8cfc3e66c21f471517ef6ddbf361f5d63c25
```

다음과 같이 결과가 나타나면 성공이다.

```bash
This node has joined the cluster and a new control plane instance was created:

* Certificate signing request was sent to apiserver and approval was received.
* The Kubelet was informed of the new secure connection details.
* Control plane (master) label and taint were applied to the new node.
* The Kubernetes control plane instances scaled up.
```

Join이 완료되었으면 마찬가지로 권한 설정을 한다.

```bash
mkdir -p $HOME/.kube;
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config;
sudo chown $(id -u):$(id -g) $HOME/.kube/config;
```

연결된 Control Plane 노드를 확인해본다.

```bash
kubectl get nodes

# NAME             STATUS   ROLES    AGE     VERSION
# controlplane-1   Ready    master   11m     v1.17.0
# controlplane-2   Ready    master   2m57s   v1.17.0
# controlplane-3   Ready    master   2m58s   v1.17.0
```

---

## Worker 노드 연결

VM을 생성한다.

```bash
vagrant up worker-1
```

호스트에 접속한다.

```bash
vagrant ssh worker-1
```

다음 과정을 따라 한다: **[kubeadm 설치](/docs/install-kubeadm.md)**

기본 설정이 완료되었으면 다음 명령어를 실행한다.

```bash
sudo kubeadm join 192.168.10.101:6443 --token jjnixg.bbg4e7amhd32vi1r \
    --discovery-token-ca-cert-hash sha256:fb36418e348299619133891bc1ae1dafee6facf8ef8962c135d6ba7b357f00e1
```

다음과 같이 결과가 나타나면 성공이다.

```bash
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.
```

controlplane 노드에서 확인해본다.

```bash
kubectl get nodes
```

```bash
NAME             STATUS     ROLES    AGE   VERSION
controlplane-1   Ready      master   21m   v1.17.0
controlplane-2   Ready      master   12m   v1.17.0
controlplane-3   Ready      master   12m   v1.17.0
worker-1         Ready      <none>   98s   v1.17.0
```

---

## 테스트

간단한 서비스 배포 [튜토리얼](/docs/tutorial.md)을 따라하여 Kubernetes HA 클러스터가 잘 구축되었는지 확인한다.









