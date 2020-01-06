# How to Kubernetes

쿠버네티스 정리

- [Kubernetes.io](https://kubernetes.io/): 쿠버네티스 공식 홈페이지
- [Documentation](https://kubernetes.io/docs/home/): 쿠버네티스 문서
- [Getting started](https://kubernetes.io/docs/setup/)

## Learning Environment

- [K8S: Installing Kubernetes with Minikube](https://kubernetes.io/docs/setup/learning-environment/minikube/)

미니큐브로 쿠버네티스 환경 연습하기

- [Minikube](/docs/minikube.md): 설치와 간단한 실습

## Production Environment

쿠버네티스 클러스터를 구축하는 방법은 여러가지다.

1. kubeadm을 사용해 클러스터 구축하기
   1. Single control-plane cluster
   2. HA clusters: Stacked etcd topology
   3. [HA clusters: External etcd topology](/docs/external-topology.md)
2. Installing Kubernetes with kops
3. Installing Kubernetes with KRIB
4. Installing Kubernetes with Kubespray

---

## 사양

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

---
