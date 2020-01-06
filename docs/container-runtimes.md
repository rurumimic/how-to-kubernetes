# Container runtimes

Pods에서 컨테이너를 실행하기 위해서, 쿠버네티스는 컨테이너 런타임을 사용한다.

다양한 런타임이 있다:

- Docker
- CRI-O
- Containerd
- frakti

## Docker

- [K8S: Docker](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker)

다음은 Ubuntu 18에서 Docker v19.03.4 설치 방법이다.

### Ubuntu 18

root 권한으로 진행한다.

```bash
sudo -Es
```

#### 저장소 설정

필요한 패키지 설치  
HTTPS를 통해 저장소를 사용할 수 있도록 설치  

```bash
apt-get update && apt-get install -y \
  apt-transport-https ca-certificates curl software-properties-common
```

도커 공식 GPG key 추가

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
```

도커 apt 저장소 추가

```bash
add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
```

#### Docker CE 설치

```bash
apt-get update && apt-get install -y \
  containerd.io=1.2.10-3 \
  docker-ce=5:19.03.4~3-0~ubuntu-$(lsb_release -cs) \
  docker-ce-cli=5:19.03.4~3-0~ubuntu-$(lsb_release -cs)
```

#### 데몬 설정

```bash
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```

```bash
mkdir -p /etc/systemd/system/docker.service.d
```

#### 도커 시작

```bash
systemctl daemon-reload
systemctl restart docker
systemctl enable docker
```

#### (Option) non-root user

```bash
sudo groupadd docker;
sudo usermod -aG docker $USER;
```