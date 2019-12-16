# Container runtimes

Pods에서 컨테이너를 실행하기 위해서, 쿠버네티스는 컨테이너 런타임을 사용한다.

다양한 런타임이 있다:

- Docker
- CRI-O
- Containerd
- frakti

## Docker

- [Docker CE: CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)

### 저장소 설정

필요한 패키지 설치

```bash
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

Complete!
```

- `yum-utils`: `yum-config-manager` 유틸 제공
- `device-mapper-persistent-data`, `lvm2`: `devicemapper` storage driver가 필요.

도커 저장소 추가

```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

repo saved to /etc/yum.repos.d/docker-ce.repo
```

### Docker CE 설치

Version 19.03.4

```bash
sudo yum -y update;
sudo yum -y install containerd.io-1.2.10 docker-ce-19.03.4 docker-ce-cli-19.03.4;
```

### /etc/docker 생성

```bash
sudo mkdir /etc/docker
```

### 데몬 설정

```bash
sudo bash -c 'cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF'
```

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
```

### 도커 시작

```bash
sudo systemctl daemon-reload
sudo systemctl start docker
sudo systemctl enable docker

Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
```

### (Option) non-root user

```bash
sudo groupadd docker;
sudo usermod -aG docker $USER;
```