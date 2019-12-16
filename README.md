# How to Kubernetes

- [Kubernetes.io](https://kubernetes.io/)
- [Documentation](https://kubernetes.io/docs/home/)

## Production Environment

- [Container runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)
- Installing Kubernetes with deployment tools
  - Bootstrapping clusters with kubeadm
  - Installing Kubernetes with kops
  - Installing Kubernetes with KRIB
  - Installing Kubernetes with Kubespray

---

## 템플릿 VM 생성

[Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

### 사양

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