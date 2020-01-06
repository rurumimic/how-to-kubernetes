# 템플릿 노드

[Vagrantfile](../template-node/Vagrantfile)

## root 권한

```bash
# root 전환
sudo su

# root 로그아웃
exit
```

## MAC 주소, product_uuid 증명

- MAC 주소
  - `ip link`
  - `ifconfig -a`
- product_uuid
  - `sudo cat /sys/class/dmi/id/product_uuid`

## iptables, nftables

`nftables`는 `kube-proxy`와 충돌이 난다. `iptables`만 활성화한다.

RHEL 8은 레거시 모드로 전환할 수 없다. 따라서 현재 kubeadm과 호환이 되지 않는다.

Debian/Ubuntu

```bash
sudo update-alternatives --set iptables /usr/sbin/iptables-legacy;
sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy;
sudo update-alternatives --set arptables /usr/sbin/arptables-legacy;
sudo update-alternatives --set ebtables /usr/sbin/ebtables-legacy;
```

## [Container Runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)

[Install Docker](install-docker.md)

## kubeadm, kubelet, kubectl 설치

### Ubuntu, Debian, HypriotOS

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
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
# 업데이트에서 제외
sudo apt-mark hold kubelet kubeadm kubectl
```

### CentOS, RHEL, Fedora

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo 
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
## SELINUX 파일 설정 변경
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# 설치
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

# kubelet 자동 실행
systemctl enable --now kubelet
```

#### Traffic Issue

Some users on RHEL/CentOS 7 have reported issues with traffic being routed incorrectly due to iptables being bypassed. You should ensure `net.bridge.bridge-nf-call-iptables` is set to `1` in your sysctl config, e.g.

```bash
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system
```

#### br_netfilter

Make sure that the `br_netfilter` module is loaded before this step. This can be done by running `lsmod | grep br_netfilter`. To load it explicitly call `modprobe br_netfilter`.

```bash
# 모듈 정보 표시: br_netfilter 22256  0
lsmod | grep br_netfilter

# 모듈 로드
modprobe br_netfilter
```

### control-plane 노드에서 kubelet이 사용하는 cgroup driver 설정

Docker를 사용하면 kubeadm은 자동으로 cgroup driver를 탐지해서 `/var/lib/kubelet/kubeadm-flags.env` 파일을 런타임 때 생성한다.

다른 CRI를 사용한다면 `/etc/default/kubelet`(CentOS, RHEL, Fedora: `/etc/sysconfig/kubelet`)를 수정한다.
이 파일은 `kubeadm init`와 `kubeadm join`를 사용할 때 쓰인다. 

```bash
KUBELET_EXTRA_ARGS=--cgroup-driver=<value>
```

사용하는 CRI가 `cgroupfs`가 아닐 때 설정한다.

kubelet을 다시 실행한다.

```bash
systemctl daemon-reload
systemctl restart kubelet
```

## Disable Swap

```bash
swapoff -a
```

## Vagrant box

```bash
vagrant package --output kube-ubuntu-001.box
vagrant box add kube-ubuntu-001 kube-ubuntu-001.box
```