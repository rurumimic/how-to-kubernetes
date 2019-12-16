# 템플릿 노드

[Vagrantfile](../template-node/Vagrantfile)

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

## Container Runtimes

- Install Docker

## kubeadm, kubelet, kubectl 설치
