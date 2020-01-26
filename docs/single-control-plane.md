# Single Control Plane Cluster

[K8S: Creating a single control-plane cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

---

## 구축 순서

1. Control Plane Node
1. Worker Nodes

## 환경 준비

- Ubuntu 16.04.6 LTS (Xenial Xerus)
- Control Plane:
  - RAM: 2GB
  - CPU: 2
- Worker x3:
  - RAM: 1GB
  - CPU: 1

## Vagrant

자세한 Vagrant 사용 방법: [rurumimic/how-to-vagrant](https://github.com/rurumimic/how-to-vagrant)

[Vagrantfile](/how-to-kubernetes/vagrant/single/Vagrantfile)을 빈 폴더에 저장한다.

- Control Plane: 2GB RAM, 2 CPU
  - 192.168.10.101
- Worker x 3
  - 192.168.10.111 ~ 113

## Control Plane

다음 과정을 따라 한다: **[kubeadm 설치](/docs/install-kubeadm.md)**

이제 kubeadm을 실행한다.

```bash
sudo kubeadm init <args>
```

명령의 인자는 필요한 것들로 채워준다.

1. (권장) single control-plane kubeadm cluster를 HA로 업그레이드할 계획이 있으면, `--control-plane-endpoint` 옵션을 사용한다.
2. [K8S: CNI, pod network add-on](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network)을 설치해야 한다. 어떤 CNI는 `--pod-network-cidr` 옵션이 필요하다.
3. (옵션) 1.14 버전부터 컨테이너 런타임이 자동으로 탐지된다. 만약 다른 컨테이너 런타임을 사용하거나 여러 컨테이너 런타임을 사용한다면, `--cri-socket` 옵션을 사용한다.
4. (옵션) kubeadm은 네트워크 인터페이스에 기본으로 설정된 게이트웨이를 사용한다. 다른 네트워크 인터페이스를 사용한다면, `--apiserver-advertise-address` 옵션을 사용한다. 

이 예제에서는 다음 명령어를 입력한다.

```bash
sudo kubeadm init \
--control-plane-endpoint=192.168.10.101 \
--pod-network-cidr=192.168.0.0/16 \
--apiserver-advertise-address=192.168.10.101
```

옵션 설명:
- `--control-plane-endpoint=192.168.10.101`: Vagrantfile에서 설정한 Control Plane 노드의 IP이다.
- `--pod-network-cidr=192.168.0.0/16`: CNI로 Calico를 사용한다.
- `--apiserver-advertise-address=192.168.10.101`: Vagrantfile에서 설정한 Control Plane 노드의 IP이다.

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

Calico를 설정한다.

```bash
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
```

다음 명령어로 CoreDNS와 Calico Pod가 실행이 잘 되는지 확인한다.

```bash
kubectl get pods --all-namespaces
```

## Worker 노드 연결

Worker 노드에 접속한다.

다음 과정을 따라 한다: **[kubeadm 설치](/docs/install-kubeadm.md)**

기본 설정이 완료되었으면 다음 명령어를 실행한다.

```bash
sudo kubeadm join 192.168.10.101:6443 --token segzp4.0bfy6219xj7fxd0s \
    --discovery-token-ca-cert-hash sha256:f5166952696d29c0d9f33f61fe7a4024d1ec2898ec28ebfc63cce38882900d86
```

Control Plane 노드에서 다음 명령을 실행한다: 

```bash
kubectl get nodes

# NAME           STATUS   ROLES    AGE     VERSION
# controlplane   Ready    master   14m     v1.17.1
# worker-1       Ready    <none>   2m31s   v1.17.1
# worker-2       Ready    <none>   35s     v1.17.1
# worker-3       Ready    <none>   31s     v1.17.1
```
