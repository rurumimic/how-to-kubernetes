# Load Balancer

로드밸런서는 간단하게 HAProxy를 사용한다.

---

## HAProxy

### Install

```bash
sudo apt-get install -y haproxy
```

### Configuration

haproxy 설정은 [아래 예제](#configuration-file)를 그대로 입력한다.

```bash
sudo vi /etc/haproxy/haproxy.cfg
```

설정 파일 검사

```bash
haproxy -c -V -f /etc/haproxy/haproxy.cfg
# Configuration file is valid
```

### Start

haproxy 설정 파일을 수정 했으면 재실행한다.

```bash
sudo systemctl restart haproxy;
sudo systemctl enable haproxy;
```

이제 `exit` 명령어로 ssh 접속을 종료한다.

---

### Configuration File

로드밸런서 `6443` 포트로 들어오는 연결을 Control Plane의 `kube-apiserver`로 보낸다. `kube-apiserver`의 기본 포트는 `6443`이다.

- frontend: kube-api
  - port: `6443`
- backend: kube-api
  - apiserver: `192.168.10.111 ~ 113: 6443`

```lua
global
  log /dev/log    local0
  log /dev/log    local1 notice
  chroot /var/lib/haproxy
  stats socket /run/haproxy/admin.sock mode 660 level admin
  stats timeout 30s
  user haproxy
  group haproxy
  daemon

  # Default SSL material locations
  ca-base /etc/ssl/certs
  crt-base /etc/ssl/private

  # Default ciphers to use on SSL-enabled listening sockets.
  # For more information, see ciphers(1SSL). This list is from:
  #  https://hynek.me/articles/hardening-your-web-servers-ssl-ciphers/
  ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS
  ssl-default-bind-options no-sslv3

defaults
  log     global
  mode    http
  option  httplog
  option  dontlognull
  timeout connect 5000
  timeout client  50000
  timeout server  50000
  errorfile 400 /etc/haproxy/errors/400.http
  errorfile 403 /etc/haproxy/errors/403.http
  errorfile 408 /etc/haproxy/errors/408.http
  errorfile 500 /etc/haproxy/errors/500.http
  errorfile 502 /etc/haproxy/errors/502.http
  errorfile 503 /etc/haproxy/errors/503.http
  errorfile 504 /etc/haproxy/errors/504.http

frontend kube-api
  bind :6443
  mode tcp
  option tcplog
  timeout client 30000
  default_backend kube-api

backend kube-api
  mode tcp
  option tcp-check 
  timeout server 30000
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
  server apiserver1 192.168.10.111:6443 check
  server apiserver2 192.168.10.112:6443 check
  server apiserver3 192.168.10.113:6443 check
```