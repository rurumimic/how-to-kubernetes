# How to Kubernetes

쿠버네티스 정리

- [Kubernetes.io](https://kubernetes.io/): 쿠버네티스 공식 홈페이지
- [Documentation](https://kubernetes.io/docs/home/): 쿠버네티스 문서
- [Getting started](https://kubernetes.io/docs/setup/): 쿠버네티스 구축 방법

---

## How to Install

쿠버네티스 클러스터를 **[구축하는 방법](/how-to-install.md)**

---

## Concepts

1. Overview
1. Cluster Architecture
1. Containers
1. Workloads
   1. Pods
   1. Controllers
      1. [ReplicaSet](docs/concepts/workloads/controllers/01.replicaset.md)
      1. [ReplicationController](docs/concepts/workloads/controllers/02.replication.controller.md)
      1. [Deployments](docs/concepts/workloads/controllers/03.deployments.md)
1. Services, Load Balancing, and Networking
1. Storage
1. Configuration
1. Security
1. Policies
1. Scheduling and Eviction
1. Cluster Administration
1. Extending Kubernetes

---

## How to Use

쿠버네티스를 **사용하는 방법**

1. [Horizontal Pod Autoscaler with minikube](docs/horizontal-pod-autoscaler.md)
   1. [HPA with a single control-plane cluster](docs/hpa-single-control-plane.md)
   2. [Autoscaling on multiple metrics and custom metrics](docs/hpa-custom-metrics.md)
