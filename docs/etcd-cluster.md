# Etcd Cluster

[K8S: External etcd nodes](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/#external-etcd-nodes)

---

## SSH 설정

etcd 클러스터에 파일을 공유하기 위해 미리 ssh를 설정한다.

`etcd-1`에서 관리자 권한으로 진행한다.

```bash
sudo -Es
```

RSA 방식으로 비대칭키를 만든다.

```bash
ssh-keygen -t rsa
# 전부 엔터
```

`/root/.ssh` 밑에 공개키와 개인키가 만들어졌다.

공개키를 복사한다.

```bash
cat /root/.ssh/id_rsa.pub

ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDECqRMIcW5O27Up70HXJegXU67LvctTGibTq3Wr5EXuTpVWstO1oWFUfV1VsiCLqWyNkFlA9KDbwo42sdl2Nr7wwxknK9BhYUamiswHs2ijr2+7mYlbxubO5+UQfhTsKxWUM3pan4y+32kd2wJIUNEoE4CrZxoP2IB7CA7OAV/06EFrumWpnNPVUOPVJBHE0Qu5kquHsZ4nLL7hfrOyvTR/g2IoANuUUhse1vZ/6I/Fgd78qnL7TL0ssSxuPtg+zo0D0iIoImFbFEu+gTIo/u5NzQAeTcax/3tDq2G28ByI+h37BUm97qFsRTpy676BagOp2DWlanIaz7GhdRtjPOF root@etcd-1
```

복사한 공개키를 나머지 두 etcd 호스트에 저장한다. `authorized_keys` 파일에 추가한다.

```bash
sudo -Es
vi /root/.ssh/authorized_keys
```

`etcd-1`에서 SSH로 나머지 두 etcd 호스트에 접속해본다.

```bash
root@etcd-1:~$ ssh -A 192.168.10.122
root@etcd-1:~$ ssh -A 192.168.10.123

# Are you sure you want to continue connecting (yes/no)?
# yes
```

root 권한도 획득해본다.

```bash
root@etcd-3:~# exit
vagrant@etcd-1:~$
```

---

## kubeadm

etcd 호스트 3 곳에서 공통적으로 관리자 권한으로 진행한다.

다음 과정을 따라 한다: **[kubeadm 설치](/docs/install-kubeadm.md)**

---

## 클러스터 구축

[K8S: Setting up the cluster](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/#setting-up-the-cluster)

etcd 호스트 3 곳에서 공통적으로 관리자 권한으로 진행한다.

---

### 우선순위 재설정

서비스 우선순위를 재설정한다.

```bash
cat << EOF > /etc/systemd/system/kubelet.service.d/20-etcd-service-manager.conf
[Service]
ExecStart=
#  Replace "systemd" with the cgroup driver of your container runtime. The default value in the kubelet is "cgroupfs".
ExecStart=/usr/bin/kubelet --address=127.0.0.1 --pod-manifest-path=/etc/kubernetes/manifests --cgroup-driver=systemd
Restart=always
EOF
```

```bash
systemctl daemon-reload
systemctl restart kubelet
```

---

### kubeadm 설정

`etcd-1` 호스트에서 진행한다.

etcd 호스트 IP를 차례대로 입력한다.

```bash
export HOST0=192.168.10.121;
export HOST1=192.168.10.122;
export HOST2=192.168.10.123;
```

임시 디렉터리를 만든다.

```bash
mkdir -p /tmp/${HOST0}/ /tmp/${HOST1}/ /tmp/${HOST2}/
```

환경 변수를 설정한다. 호스트 이름도 설정한다.

```bash
ETCDHOSTS=(${HOST0} ${HOST1} ${HOST2});
NAMES=("etcd-1" "etcd-2" "etcd-3");
```

다음 스크립트를 붙여넣는다.

```bash
for i in "${!ETCDHOSTS[@]}"; do
HOST=${ETCDHOSTS[$i]}
NAME=${NAMES[$i]}
cat << EOF > /tmp/${HOST}/kubeadmcfg.yaml
apiVersion: "kubeadm.k8s.io/v1beta2"
kind: ClusterConfiguration
etcd:
    local:
        serverCertSANs:
        - "${HOST}"
        peerCertSANs:
        - "${HOST}"
        extraArgs:
            initial-cluster: ${NAMES[0]}=https://${ETCDHOSTS[0]}:2380,${NAMES[1]}=https://${ETCDHOSTS[1]}:2380,${NAMES[2]}=https://${ETCDHOSTS[2]}:2380
            initial-cluster-state: new
            name: ${NAME}
            listen-peer-urls: https://${HOST}:2380
            listen-client-urls: https://${HOST}:2379
            advertise-client-urls: https://${HOST}:2379
            initial-advertise-peer-urls: https://${HOST}:2380
EOF
done
```

etcd 클러스터 ca 파일을 생성한다.

```bash
kubeadm init phase certs etcd-ca
```

다음 명령어들을 차례대로 실행한다.

```bash
kubeadm init phase certs etcd-server --config=/tmp/${HOST2}/kubeadmcfg.yaml;
kubeadm init phase certs etcd-peer --config=/tmp/${HOST2}/kubeadmcfg.yaml;
kubeadm init phase certs etcd-healthcheck-client --config=/tmp/${HOST2}/kubeadmcfg.yaml;
kubeadm init phase certs apiserver-etcd-client --config=/tmp/${HOST2}/kubeadmcfg.yaml;
cp -R /etc/kubernetes/pki /tmp/${HOST2}/;
# cleanup non-reusable certificates
find /etc/kubernetes/pki -not -name ca.crt -not -name ca.key -type f -delete;

kubeadm init phase certs etcd-server --config=/tmp/${HOST1}/kubeadmcfg.yaml;
kubeadm init phase certs etcd-peer --config=/tmp/${HOST1}/kubeadmcfg.yaml;
kubeadm init phase certs etcd-healthcheck-client --config=/tmp/${HOST1}/kubeadmcfg.yaml;
kubeadm init phase certs apiserver-etcd-client --config=/tmp/${HOST1}/kubeadmcfg.yaml;
cp -R /etc/kubernetes/pki /tmp/${HOST1}/;
find /etc/kubernetes/pki -not -name ca.crt -not -name ca.key -type f -delete;

kubeadm init phase certs etcd-server --config=/tmp/${HOST0}/kubeadmcfg.yaml;
kubeadm init phase certs etcd-peer --config=/tmp/${HOST0}/kubeadmcfg.yaml;
kubeadm init phase certs etcd-healthcheck-client --config=/tmp/${HOST0}/kubeadmcfg.yaml;
kubeadm init phase certs apiserver-etcd-client --config=/tmp/${HOST0}/kubeadmcfg.yaml;
# No need to move the certs because they are for HOST0

# clean up certs that should not be copied off this host
find /tmp/${HOST2} -name ca.key -type f -delete;
find /tmp/${HOST1} -name ca.key -type f -delete;
```

먼저 etcd-2 호스트로 파일을 전송한다.

```bash
scp -r /tmp/${HOST1}/* root@${HOST1}:
ssh root@${HOST1}

root@HOST1 $ mv /root/pki /etc/kubernetes/
root@HOST1 $ exit
```

`etcd-3` 호스트에도 전송한다.

```bash
scp -r /tmp/${HOST2}/* root@${HOST2}:
ssh root@${HOST2}

root@HOST2 $ mv /root/pki /etc/kubernetes/
root@HOST2 $ exit
```

각 etcd 호스트에서 다음 명령어를 실행한다.

```bash
root@HOST0 $ kubeadm init phase etcd local --config=/tmp/${HOST0}/kubeadmcfg.yaml
root@HOST1 $ kubeadm init phase etcd local --config=/root/kubeadmcfg.yaml
root@HOST2 $ kubeadm init phase etcd local --config=/root/kubeadmcfg.yaml
```

---

### 클러스터 확인

[K8S: Endpoint Health](https://github.com/etcd-io/etcd/tree/master/etcdctl#endpoint-health)

Etcd 도커 이미지 버전을 확인한다.

```bash
docker images
```

```bash
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/etcd     3.4.3-0             303ce5db0e90        2 months ago        288MB
k8s.gcr.io/pause    3.1                 da86e6ba6ca1        2 years ago         742kB
```

- `${ETCD_TAG}`: etcd version: `v3.4.3`
- `${HOST0}`: 호스트 IP: `192.168.10.121`

```bash
ETCD_TAG=v3.4.3;
HOST0=192.168.10.121;

docker run --rm -it \
--net host \
-v /etc/kubernetes:/etc/kubernetes quay.io/coreos/etcd:${ETCD_TAG} etcdctl \
--cert /etc/kubernetes/pki/etcd/peer.crt \
--key /etc/kubernetes/pki/etcd/peer.key \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--endpoints https://${HOST0}:2379 endpoint --cluster health
```

Etcd 클러스터 구축을 완료했다.

```bash
https://192.168.10.122:2379 is healthy: successfully committed proposal: took = 19.321318ms
https://192.168.10.121:2379 is healthy: successfully committed proposal: took = 29.940281ms
https://192.168.10.123:2379 is healthy: successfully committed proposal: took = 30.706686ms
```