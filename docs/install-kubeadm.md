# kubeadm 설치

쿠버네티스 노드에 공통으로 설정하는 과정을 담았다.

[K8S: Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

## Disable Swap

kubelet가 잘 동작하기 위해서는 swap을 비활성화 해야 한다.

```bash
swapoff -a
```

메모리 상태를 확인해본다.

```bash
free -m
```

```bash
              total        used        free      shared  buff/cache   available
Mem:           1993          86        1313           0         593        1750
Swap:             0           0           0
```

## MAC 주소와 product_uuid 확인

이 과정은 건너 뛴다. 신경 안 써도 된다. 가상 머신이 자동으로 유일한 값으로 설정한다.

```bash
ip link
```

```bash
sudo cat /sys/class/dmi/id/product_uuid
```

## 네트워크 아답터 확인

네트워크 아답터가 여러 개일 경우, 자칫하면 안 된다.  
적절한 네트워크 아답터를 게이트웨이로 설정해야 한다.

[작성 중]

## nftable 비활성화

`nftables`는 `kube-proxy`와 충돌이 난다. `iptables`만 활성화한다.

Vagrant Box로 설치한 Ubuntu는 nftable을 사용하지 않는 것 같으니, 넘어간다.

## 포트

넘어가자.

## Runtime 설치

- [K8S: Installing runtime](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime)
- [K8S: Container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)

쿠버네티스 1.6.0 버전부터 Container Runtime Interface를 기본으로 사용한다.

Runtime 우선 순위: Docker > containerd > CRI-O

런타임으로 Docker를 사용하자.

[컨테이너 런타임 설치 방법](/docs/container-runtimes.md)

## kubeadm, kubelet, kubectl 설치

컨테이너 런타임과 마찬가지로 모든 노드에 설치해야 하는 것이다.

- kubeadm: 클러스터 실행 툴
- kubelet: pod과 container를 실행하는 컴포넌트
- kubectl: 클러스터와 통신하는 명령 툴

각각 어울리는 버전이 있다. 다음을 참조한다.

- Kubernetes [version and version-skew policy](https://kubernetes.io/docs/setup/release/version-skew-policy/)
- Kubeadm-specific [version skew policy](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#version-skew-policy)

하지만 다음대로 설치하면 문제없다.

root 권한으로 진행한다. (`sudo -Es`)

`apt-transport-https`, `curl`는 이미 도커를 설치할 때 설치했다.

GPG key 추가

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

저장소 추가

```bash
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

설치

```bash
apt-get update && apt-get install -y kubelet kubeadm kubectl
```

업데이트에서 제외

```bash
apt-mark hold kubelet kubeadm kubectl
```

이제부터 kubelet은 kubeadm이 어떤 명령을 내릴지 기다리면서 계속 재시작된다.

## cgroup driver 설정

Control Plane에서 kubelet이 사용하는 cgroup driver 설정을 해야 한다.

도커를 사용하면 자동으로 설정된다.

넘어간다.

## Control Plane의 Config Image

Control Plane에서 config 이미지를 미리 다운 받는다.

```bash
kubeadm config images pull
```