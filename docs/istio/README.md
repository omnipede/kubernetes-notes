# Istio

## Service mesh
기존의 MSA 아키텍처에서의 호출이 직접 호출방식이였다면 ```service mesh``` 에서의 호출은 서비스에 붙은 sidecar proxy 끼리 이루어지게 된다.

| 기존 구조 | Service mesh |
| --- | --- |
| ![image](https://user-images.githubusercontent.com/41066039/119587697-2d117000-be0a-11eb-905a-d6f63c462322.png)| ![image](https://user-images.githubusercontent.com/41066039/119587745-46b2b780-be0a-11eb-88bd-5d2ff9a125c3.png)|

이러한 service mesh 구성은 개발자로 하여금 서비스 호출에 대한 부담을 줄이고 업무 로직에 집중할 수 있게 해준다.
하지만 service 가 많아질 수록 이에 정비례하여 proxy 의 수도 같이 증가한다. 많은 proxy 들은 중앙 집중화된 컨트롤러에 의해 관리된다.

![image](https://user-images.githubusercontent.com/41066039/119587810-6944d080-be0a-11eb-9f2b-b4a3076d75b3.png)


프록시들로 이루어져 트래픽을 설정값에 따라 컨트롤하는 부분을 Data Plane 이라고 하고,
프록시들에 설정값을 전달하고 관리하는 컨트롤러 역할을 Control Plane 이라고 한다.

```Istio``` 는 이러한 ```service mesh``` 아키텍처에 대한 구현체의 일종이다.

## Istio 설치

* 먼저 istioctl binary 를 설치해야 한다.

```
$ curl -L https://git.io/getLatestIstio | sh -
$ cd istio-*
$ sudo mv -v bin/istioctl /usr/local/bin/
```

* 설치된 istioctl 을 이용해서 istio k8s 리소스를 설치한다.

```
$ istioctl install
$ kubectl get deploy -n istio-system
```

* Envoy sidecar injection 설정
Application 배포 시 istio 가 자동으로 envoy sidecar proxy 를 삽입하도록 namespace label 을 추가한다. 
```
$ kubectl label namespace default istio-injection=enabled
$ kubectl get namespaces --show-labels
```

## Istio book info 예제

istioctl binary 설치 시 생성된 ```istio-*``` 디렉토리 내부 ```samples``` 디렉토리에 istio 예제가 있다.
이 중 ```bookinfo``` 예제를 살펴보자.

![image](https://user-images.githubusercontent.com/41066039/119610476-4202f900-be34-11eb-8599-f86e49930ca3.png)

이 예제는 ```product page```, ```reviews```, ```details```, ```raitings``` 서비스로 이루어져 있다. 
```product page``` 를 통해 책 상세 정보, 리뷰, 별점 등을 확인하는 형태다. 

### 기본 세팅
먼저 기본 서비스 리소스를 생성한다.
```
$ kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
```

배포된 pod 들을 살펴보면 envoy proxy 가 삽입되어서 container ready 개수가 ```2/2``` 로 나타난다.

```
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
details-v1-79f774bdb9-488r7       2/2     Running   0          5m51s
productpage-v1-6b746f74dc-mzvn7   2/2     Running   0          5m51s
ratings-v1-b6994bb9-5qv5q         2/2     Running   0          5m51s
reviews-v1-545db77b95-7t9vc       2/2     Running   0          5m50s
reviews-v2-7bf8c9648f-zqwn9       2/2     Running   0          5m51s
reviews-v3-84779c7bbc-grfs8       2/2     Running   0          5m51s
```

현재 서비스 리소스는 ClusterIP 타입밖에 없다. 
```
$ kubectl get svc
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
details       ClusterIP   10.103.248.163   <none>        9080/TCP   24m
kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP    12d
productpage   ClusterIP   10.96.174.115    <none>        9080/TCP   24m
ratings       ClusterIP   10.107.168.86    <none>        9080/TCP   24m
reviews       ClusterIP   10.111.207.129   <none>        9080/TCP   24m
```

외부에서 접근이 불가능한 상태이므로 리소스를 추가로 생성해서 외부 접근이 가능하게 만들어야 한다.
Istio 를 사용하여 이 문제를 해결해보자. Istio gateway 를 생성한 뒤 gateway 를 통해 서비스할 호스트를 virtual service 로 등록한다. 
그림으로 나타내면 다음과 같다.
![image](https://user-images.githubusercontent.com/41066039/119611443-abcfd280-be35-11eb-8e47-ef62606f26fa.png)

명령어로 book info gateway 를 생성한다.
```
$ kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
$ istioctl analyze
```

Book info gateway 는 이렇게 구성되어있다.
```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
```
Selector 는 gateway controller 를 지정하는 역할을 한다. 예제에서는 기본 gateway controller 를 사용하고 있다. 
HTTP 프로토콜을 80 포트에서 받고 있고, 모든 도메인 (hosts) 에 대한 요청을 받고 있다.

그리고 gateway 에 연동하는 virtual service 는 이렇게 구성되어있다.
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```
```bookinfo-gateway``` 로부터 들어오는 요청을 받고 있고 모든 도메인에 대해 요청을 받고 있다. 
그리고 ```/productpage``` prefix 가 붙은 경우 ```productpage``` 서비스 리소스로 라우팅하고 있다.
브라우저를 이용해서 확인해보자.

![image](https://user-images.githubusercontent.com/41066039/119621352-f99e0800-be40-11eb-87c4-052fc7bc442b.png)

### Traffic 제어

```product page``` 서비스에서 ```reviews``` 서비스 v1, v2, v3 쪽으로 요청을 보내고 있다. 
여기서 ```reviews``` 서비스 v2, v3 에게 각각 50:50 비율로 traffic 이 가도록 해보자.

먼저 destination rule 을 생성해서 ```reviews``` 서비스의 subset 을 설정한다.
```
$ kubectl apply -f samples/bookinfo/networking/destination-rule-reviews.yaml
```

그리고 ```product page``` --> ```reviews``` 사이에 새로운 Virtual service 를 끼워넣는다.
Virtual service 의 설정은 다음과 같다.

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v2
      weight: 50
    - destination:
        host: reviews
        subset: v3
      weight: 50
```

```reviews``` 서비스에 기존에 설정했던 subset v2, v3 으로 가중치 50:50 을 주고 있다.
설정한 Virtual service 를 적용하자.

```
$ kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
```

브라우저를 통해 ```/productpage``` 로 접속해보자. 접속한 뒤 새로 고침을 하면 ```reviews``` 서비스의 v2, v3 를 번갈아가며 호출하는 것을 확인할 수 있다.

### 예제 삭제
```cleanup.sh``` 명령어로 예제 리소스를 전부 삭제한다.

```
$ sh ./samples/bookinfo/platform/kube/cleanup.sh
```
