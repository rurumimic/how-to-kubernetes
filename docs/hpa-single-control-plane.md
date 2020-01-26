# HPA with a single control-plane cluster

## Metrics server

[kubernetes-sigs
/
metrics-server](https://github.com/kubernetes-sigs/metrics-server)

```bash
git clone https://github.com/kubernetes-sigs/metrics-server.git
```

```bash
kubectl create -f metrics-server/deploy/1.8+/

# clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
# clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
# rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
# apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
# serviceaccount/metrics-server created
# deployment.apps/metrics-server created
# service/metrics-server created
# clusterrole.rbac.authorization.k8s.io/system:metrics-server created
# clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
```

```bash
kubectl get pods -A

# NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
# kube-system   calico-kube-controllers-5c45f5bd9f-w5jfd   1/1     Running   0          20m
# kube-system   calico-node-2qrsz                          1/1     Running   0          20m
# kube-system   calico-node-9lpcc                          1/1     Running   0          9m7s
# kube-system   calico-node-9p8fh                          1/1     Running   0          9m11s
# kube-system   calico-node-wfj5m                          1/1     Running   0          11m
# kube-system   coredns-6955765f44-4krt5                   1/1     Running   0          22m
# kube-system   coredns-6955765f44-tbmgc                   1/1     Running   0          22m
# kube-system   etcd-controlplane                          1/1     Running   0          22m
# kube-system   kube-apiserver-controlplane                1/1     Running   0          22m
# kube-system   kube-controller-manager-controlplane       1/1     Running   0          22m
# kube-system   kube-proxy-hdbvx                           1/1     Running   0          22m
# kube-system   kube-proxy-hlwnc                           1/1     Running   0          11m
# kube-system   kube-proxy-tzkgs                           1/1     Running   0          9m7s
# kube-system   kube-proxy-vk2md                           1/1     Running   0          9m11s
# kube-system   kube-scheduler-controlplane                1/1     Running   0          22m
# kube-system   metrics-server-795b774c76-vvczl            1/1     Running   0          28s
```

```bash
kubectl describe pod metrics-server-795b774c76-fn5mj -n kube-system
```

```bash
kubectl run php-apache --image=k8s.gcr.io/hpa-example --requests=cpu=200m --limits=cpu=500m --expose --port=80
```

```bash
kubectl autoscale deployment php-apache --cpu-percent=20 --min=1 --max=10
```

```bash
kubectl get hpa

# NAME         REFERENCE               TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
# php-apache   Deployment/php-apache   <unknown>/20%   1         10        0          4s
```

```bash
kubectl describe hpa

# Name:                                                  php-apache
# Namespace:                                             default
# Labels:                                                <none>
# Annotations:                                           <none>
# CreationTimestamp:                                     Sun, 19 Jan 2020 07:21:14 +0000
# Reference:                                             Deployment/php-apache
# Metrics:                                               ( current / target )
#   resource cpu on pods  (as a percentage of request):  <unknown> / 20%
# Min replicas:                                          1
# Max replicas:                                          10
# Deployment pods:                                       1 current / 0 desired
# Conditions:
#   Type           Status  Reason                   Message
#   ----           ------  ------                   -------
#   AbleToScale    True    SucceededGetScale        the HPA controller was able to get the target's current scale
#   ScalingActive  False   FailedGetResourceMetric  the HPA was unable to compute the replica count: unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server is currently unable to handle the request (get pods.metrics.k8s.io)
# Events:
#   Type     Reason                        Age   From                       Message
#   ----     ------                        ----  ----                       -------
#   Warning  FailedGetResourceMetric       2s    horizontal-pod-autoscaler  unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server is currently unable to handle the request (get pods.metrics.k8s.io)
#   Warning  FailedComputeMetricsReplicas  2s    horizontal-pod-autoscaler  invalid metrics (1 invalid out of 1), first error is: failed to get cpu utilization: unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server is currently unable to handle the request (get pods.metrics.k8s.io)
```

```bash
kubectl edit deploy -n kube-system metrics-server

# args:
# - --kubelet-insecure-tls
# - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
```

```bash
kubectl describe hpa

# Name:                                                  php-apache
# Namespace:                                             default
# Labels:                                                <none>
# Annotations:                                           <none>
# CreationTimestamp:                                     Sun, 19 Jan 2020 07:21:14 +0000
# Reference:                                             Deployment/php-apache
# Metrics:                                               ( current / target )
#   resource cpu on pods  (as a percentage of request):  <unknown> / 20%
# Min replicas:                                          1
# Max replicas:                                          10
# Deployment pods:                                       1 current / 0 desired
# Conditions:
#   Type           Status  Reason                   Message
#   ----           ------  ------                   -------
#   AbleToScale    True    SucceededGetScale        the HPA controller was able to get the target's current scale
#   ScalingActive  False   FailedGetResourceMetric  the HPA was unable to compute the replica count: unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server is currently unable to handle the request (get pods.metrics.k8s.io)
# Events:
#   Type     Reason                        Age                     From                       Message
#   ----     ------                        ----                    ----                       -------
#   Warning  FailedComputeMetricsReplicas  3m58s (x12 over 6m45s)  horizontal-pod-autoscaler  invalid metrics (1 invalid out of 1), first error is: failed to get cpu utilization: unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server is currently unable to handle the request (get pods.metrics.k8s.io)
#   Warning  FailedGetResourceMetric       101s (x21 over 6m45s)   horizontal-pod-autoscaler  unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server is currently unable to handle the request (get pods.metrics.k8s.io)
```