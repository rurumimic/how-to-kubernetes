# How to Install

쿠버네티스 클러스터 구축 방법

## Learning Environment

- [K8S: Installing Kubernetes with Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)

미니큐브로 쿠버네티스 환경 연습하기

- [Minikube](/docs/minikube.md): 설치와 간단한 실습

## Production Environment

쿠버네티스 클러스터를 구축하는 방법은 여러가지다.

1. kubeadm을 사용해 클러스터 구축하기
   1. [Single control-plane cluster](/docs/single-control-plane.md)
   2. HA clusters: Stacked etcd topology
   3. [HA clusters: External etcd topology](/docs/external-topology.md)
2. Installing Kubernetes with kops
3. Installing Kubernetes with KRIB
4. Installing Kubernetes with Kubespray

### Stacked와 External Topology

Stacked Topology는 설정이 간단하고, 복제 관리도 간단하다. 하지만 한 노드가 다운되면 etcd 멤버와 control plane 인스턴스가 모두 손실되고 자료가 손상된다.

External Topology는 제어부와 데이터 스토리지를 분리했다. control plane 호스트나 etcd 멤버를 잃어도 영향이 적은 안정적인 HA 설정이 가능하다. 하지만 Stacked HA Topology보다 호스트가 배로 필요하다.

---

## 최소 사양

- OS
  - Ubuntu 16.04+
  - Debian 9+
  - CentOS 7
  - Red Hat Enterprise Linux (RHEL) 7
  - Fedora 25+
  - HypriotOS v1.0.1+
  - Container Linux (tested with 1800.6.0)
- 2 GB 이상 RAM
- 2 CPU 이상
- 원활한 네트워크 연결
- 유일한 hostname, MAC 주소, product_uuid
- 특정 포트 개방
- **반드시** Swap disabled
