# Control Plane 2, 3번

## config images pull

```bash
sudo kubeadm config images pull

W0106 16:09:11.816872    5664 validation.go:28] Cannot validate kube-proxy config - no validator is available
W0106 16:09:11.816960    5664 validation.go:28] Cannot validate kubelet config - no validator is available
[config/images] Pulled k8s.gcr.io/kube-apiserver:v1.17.0
[config/images] Pulled k8s.gcr.io/kube-controller-manager:v1.17.0
[config/images] Pulled k8s.gcr.io/kube-scheduler:v1.17.0
[config/images] Pulled k8s.gcr.io/kube-proxy:v1.17.0
[config/images] Pulled k8s.gcr.io/pause:3.1
[config/images] Pulled k8s.gcr.io/etcd:3.4.3-0
[config/images] Pulled k8s.gcr.io/coredns:1.6.5
```

## kubeadm join

```bash
sudo kubeadm join 192.168.10.101:6443 --token jjnixg.bbg4e7amhd32vi1r \
>     --discovery-token-ca-cert-hash sha256:fb36418e348299619133891bc1ae1dafee6facf8ef8962c135d6ba7b357f00e1 \
>     --control-plane --certificate-key 06c4a3bb5a5525014c5c358991de8cfc3e66c21f471517ef6ddbf361f5d63c25

[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[preflight] Running pre-flight checks before initializing the new control plane instance
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[download-certs] Downloading the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [controlplane-2 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.2.15 192.168.10.101]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[certs] Using the existing "sa" key
[kubeconfig] Generating kubeconfig files
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
W0106 16:10:53.204429    6234 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0106 16:10:53.213215    6234 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0106 16:10:53.215573    6234 manifests.go:214] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[check-etcd] Skipping etcd check in external mode
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.17" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...
[control-plane-join] using external etcd - no local stacked instance added
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[mark-control-plane] Marking the node controlplane-2 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node controlplane-2 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]

This node has joined the cluster and a new control plane instance was created:

* Certificate signing request was sent to apiserver and approval was received.
* The Kubelet was informed of the new secure connection details.
* Control plane (master) label and taint were applied to the new node.
* The Kubernetes control plane instances scaled up.


To start administering your cluster from this node, you need to run the following as a regular user:

        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config

Run 'kubectl get nodes' to see this node join the cluster.
```

## get nodes

```bash
kubectl get nodes

NAME             STATUS   ROLES    AGE     VERSION
controlplane-1   Ready    master   11m     v1.17.0
controlplane-2   Ready    master   2m57s   v1.17.0
controlplane-3   Ready    master   2m58s   v1.17.0
```