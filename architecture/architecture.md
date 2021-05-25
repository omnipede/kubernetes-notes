# 쿠버네티스 아키텍처

![image](https://user-images.githubusercontent.com/41066039/119455694-d017aa80-bd74-11eb-88ec-b44492956bde.png)

참고로 대부분의 아키텍처 구성요소들은 
```
$ kubectl get all -n kube-system
```
로 확인이 가능하다.

## Master node (Control Plane)
클러스터를 관리, 제어하는 요소와 클러스터 상태에 관한 데이터를 갖고 있다. Master node 는 다음 구성요소를 포함한다.

### API server
쿠버네티스에 대한 모든 기능들을 REST API 로 제공하고 그에 대한 명령을 처리한다.

### Scheduler
Pod, service 등 리소스를 적절한 worker node 에게 할당하는 스케쥴러

### etcd
클러스터 상태 및 구성을 지속적으로 저장하는 분산 데이터 스토리지

### Controller manager
여러 컨트롤러 (Service controller, Node controller, Volume controller) 를 배포, 관리하는 역할을 한다.

### DNS
Pod, service 등의 리소스는 IP 를 배정받는다. 리소스들은 동적으로 생성되므로 IP 주소가 매번 바뀐다. IP 주소와 DNS 이름을 DNS 서버에 등록 후
DNS 이름을 기반으로 하여 리소스에 접근할 수 있도록 한다 (서비스 디스커버리 방식). 

## Worker node
워커 노트는 어플리케이션의 실행을 담당하는 시스템이다. Worker node 는 다음 구성요소를 포함한다.

### Kubelet
Master node 의 API server 와 통신하고 worker node 상에서 컨테이너를 관리 및 실행한다. 또한 노드의 상태를 master 로 전달하는 역할을 한다.

### kube-proxy
Worker node 로 들어오는 네트워크 트래픽을 적절한 컨테이너로 라우팅하고 worker node 와 master node 간의 네트워크 통신을 관리한다.
