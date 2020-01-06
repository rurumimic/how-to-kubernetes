# Install Minikube

[K8S: Install Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

## 가상화 지원 확인

macOS에서 VMX 기능이 지원되는지 확인한다.

```bash
sysctl -a | grep -E --color 'machdep.cpu.features|VMX'
```

## 미니큐브 설치

쿠버네티스를 사용하기 위해서는 다음 3가지를 설치해야 한다.

1. kubectl: command-line tool. 쿠버네티스 명령 툴.
2. Hypervisor: [VirtualBox](https://www.virtualbox.org/wiki/Downloads)를 설치한다.
3. Minikube

### kubectl 설치

[K8S: Install kubectl on macOS](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-macos)

macOS에서는 3가지 방법 중 하나로 kubectl를 설치한다.

1. kubectl binary를 직접 설치
2. Homebrew를 사용한 설치
3. Macports를 사용한 설치

간단하게 Homebrew로 설치한다.

```bash
brew install kubectl
```

설치 완료 후 버전을 확인한다.

```bash
kubectl version
```

옵션: [kubectl 설정](https://kubernetes.io/docs/tasks/tools/install-kubectl/#optional-kubectl-configurations)

- [쉘 자동완성 설정](https://kubernetes.io/docs/tasks/tools/install-kubectl/#enabling-shell-autocompletion)

### Minikube 설치

[K8S: Install Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/#install-minikube)

1. Homebrew를 사용한 설치
2. binary를 직접 설치

간단하게 Homebrew로 설치한다.

```bash
brew install minikube
```

## 설치 확인

[K8S: Confirm Installation](https://kubernetes.io/docs/tasks/tools/install-minikube/#confirm-installation)

미니큐브를 처음 시작할 때는 다음 명령어를 실행한다.

```bash
minikube start --vm-driver=virtualbox
```

실행이 완료되면 클러스터 상태를 확인한다.

```bash
minikube status
```

```bash
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

클러스터를 정지한다.

```bash
minikube stop
```

## 로컬 상태 초기화

[K8S: Clean up local state](https://kubernetes.io/docs/tasks/tools/install-minikube/#cleanup-local-state)

미니큐브를 다시 실행한다.

```bash
minikube start
```

만약 `machine does not exist`라고 나오면, 미니큐브 상태를 초기화하고 다시 시작한다.

```bash
minikube delete
```