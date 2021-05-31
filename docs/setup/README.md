# 설치 및 클러스터 구성

## 설치 방법

### 1. 인스턴스 준비
Master node 와 worker node 로 구성된 클러스터를 구성하기 위해 다수의 인스턴스를 준비해야한다. 
AWS, GCP 등 클라우드 인스턴스를 준비해도 되고 VM 환경으로 준비해도 된다. 
클라우드 환경은 요금이 발생할 수 있기 때문에 본 문서에서는 VM 환경으로 인스턴스를 준비했다. 
macOS ```multipass``` 를 이용해서 아래와 같이 구성했다.

```
$ multipass list
Name                    State             IPv4             Image
master-node             Running           192.168.64.4     Ubuntu 20.04 LTS
worker-node-1           Running           192.168.64.5     Ubuntu 20.04 LTS
worker-node-2           Running           192.168.64.6     Ubuntu 20.04 LTS
```

단, 인스턴스는 쿠버네티스를 설치하기 위해 최소
* 2 CPU
* 2GB memory
* 5GB disk  

이상의 스팩은 갖추고 있어야 한다.

### 2. Docker 설치

```
$ curl -sSL get.docker.com | sh && \
  sudo usermod -aG docker $USER
```

```docker ps``` 로 docker 가 정상 작동하는지 확인. Permission denied 가 발생하면 로그아웃 후 재접속.

### 3. Kubeadm 설치

```Kubeadm``` 은 클러스터 구성 시 사용하는 프로그램이다. 다음 명령어로 설치해주자

```
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - && \
  echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list && \
  sudo apt-get update -q && \
  sudo apt-get install -qy kubeadm
```

### 4. Swap memory 비활성화

```
$ sudo swapoff -a
```

## 클러스터 구성 방법

### Master node

```
$ sudo kubeadm init --pod-network-cidr=$(YOUR_CIDR)
```

위 명령어를 실행시키면 ```kubeadm``` 은 콘솔에 두 가지 명령어를 출력시킨다. 이 명령어들을 기억하고 있어야 한다.

이 중 다음 명령어를 master node 에서 실행시켜준다.

```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

그리고 ```kubectl get pods -n kube-system``` 으로 쿠버네티스 필수 pod 이 전부 ```Ready``` 상태인지 확인한다.
만약 일부 pod 이 비활성화된 상태라면 Flannel CNI pod 을 설치한다.

```
$ sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

## Worker node
```kubeadm init``` 명령어 실행 시 출력된 명령어 중 다음 명령어를 모든 worker node 상에서 실행 시킨다.

```
$ sudo kubeadm join $(YOUR_MASTER_NODE_IP) --token 1liumq.jz56jwd81qqxfplb \
    --discovery-token-ca-cert-hash sha256:<TOKEN-YOU-HAVE-FROM-ABOVE>
```

그리고 ```kubectl get nodes``` 로 모든 노드가 정상적으로 구성되었는지 확인한다.

```
NAME            STATUS   ROLES                  AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
master-node     Ready    control-plane,master   2d16h   v1.21.1   192.168.64.4   <none>        Ubuntu 20.04.2 LTS   5.4.0-73-generic   docker://20.10.6
worker-node-1   Ready    <none>                 2d15h   v1.21.1   192.168.64.5   <none>        Ubuntu 20.04.2 LTS   5.4.0-73-generic   docker://20.10.6
worker-node-2   Ready    <none>                 2d15h   v1.21.1   192.168.64.6   <none>        Ubuntu 20.04.2 LTS   5.4.0-73-generic   docker://20.10.6
```

## 테스트

[deployment.yaml](./deployment.yaml) 을 이용해서 테스트를 해보자.

```kubectl apply -f deployment.yaml```

배포 후 생성된 ```pod``` 리소스는 다음과 같다.

```
NAME                            READY   STATUS    RESTARTS   AGE     IP            NODE            NOMINATED NODE   READINESS GATES
whoami-deploy-85d8fd84b-4h88d   1/1     Running   1          2d15h   192.168.2.6   worker-node-2   <none>           <none>
whoami-deploy-85d8fd84b-7jd88   1/1     Running   1          2d15h   192.168.1.6   worker-node-1   <none>           <none>
whoami-deploy-85d8fd84b-8gcsw   1/1     Running   1          2d15h   192.168.1.7   worker-node-1   <none>           <none>
whoami-deploy-85d8fd84b-bcmnc   1/1     Running   1          2d15h   192.168.1.5   worker-node-1   <none>           <none>
whoami-deploy-85d8fd84b-ms248   1/1     Running   1          2d15h   192.168.2.5   worker-node-2   <none>           <none>
whoami-deploy-85d8fd84b-qhrdn   1/1     Running   1          2d15h   192.168.2.7   worker-node-2   <none>           <none>
```

2 개의 worker node 에 균등하게 pod 이 배포된 것을 확인할 수 있다.

## 참고자료
* [[추천] Kubernetes Multi-Node Cluster with MultiPass on Ubuntu 18.04 Desktop](https://medium.com/platformer-blog/kubernetes-multi-node-cluster-with-multipass-on-ubuntu-18-04-desktop-f80b92b1c6a7)
* [Kubernetes Cluster Installation](https://velog.io/@dry8r3ad/Kubernetes-Cluster-Installation)
* [Kubeadm 으로 k8s 구성](https://velog.io/@seunghyeon/Kubeadm%EC%9C%BC%EB%A1%9C-K8S-%EA%B5%AC%EC%84%B1)
