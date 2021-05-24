# Service

Kubernetes 는 여러 종류의 service 를 제공해준다. Service type 은 다음과 같다

* ClusterIP
* NodePort 
* LoadBalancer
* ExternalName

k8s 의 Service 는 Pod 집합에서 실행중인 애플리케이션을 네트워크 서비스로 노출하는 추상화기능을 수행한다.

## ClusterIP

서비스에 클러스터 IP (내부 IP) 를 부여한다. k8s 클러스터 내에서만 서비스에 도달할 수 있다. 외부에서 
[예제](clusterip.yaml) 을 실행시켜보면 다음과 같이 object 가 생성된다.

```
NAME                          READY   STATUS    RESTARTS   AGE
pod/redis-b6fd45ccc-4rstl     1/1     Running   0          9m59s
pod/redis-b6fd45ccc-c97tq     1/1     Running   0          9m59s
pod/redis-b6fd45ccc-tkbzw     1/1     Running   0          20m
pod/whoami-676d78bbf8-rsb78   1/1     Running   0          3s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    6d5h
service/redis        ClusterIP   10.104.189.39   <none>        6379/TCP   20m

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/redis    3/3     3            3           20m
deployment.apps/whoami   1/1     1            1           3s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/redis-b6fd45ccc     3         3         3       20m
replicaset.apps/whoami-676d78bbf8   1         1         1       3s
```

```service/redis``` 라는 ClusterIP object 가 생성되었고 해당 service object 에 할당된 ip 를 확인할 수 있다. 
해당 IP 를 통해 redis pod 에 접속할 수 있다. 만약 redis pod 이 다수 실행중이라면 해당 IP 에 접근했을 때 마치 load balancer 처럼 여러개의 pod 에 번갈아가면서 접근하게 된다. 

```service/redis``` 를 describe 하면 redis pod 의 endpoints 를 확인할 수 있다.

```
Name:              redis
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          service=redis,type=db
Type:              ClusterIP
IP:                10.104.189.39
Port:              <unset>  6379/TCP
TargetPort:        6379/TCP
Endpoints:         10.1.0.56:6379,10.1.0.57:6379,10.1.0.58:6379
Session Affinity:  None
Events:            <none>
```

## NodePort
클러스터 IP 뿐 아니라 모든 k8s 노드의 IP, 포트를 통해서 서비스에 접근이 가능하게 한다.
[예제](./nodeport.yaml) 를 실행하면 NodePort service object 가 실행된다.

```
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          6d5h
service/whoami       NodePort    10.99.135.11   <none>        4567:31305/TCP   3s
```

```31305``` 포트와 ```4567``` 포트가 맵핑 되어있는 것을 알 수 있다. ```31305``` 포트로 접근하면 ```4567``` 포트의 프로세스로 접근할 수 있다.

## LoadBalancer
AWS, GCP,과 같은 Public Cloud 서비스에서 지원하는 쿠버네티스 로드밸런서 장비를 사용하여 서비스를 외부에 노출시킨다.
[예제](./loadbalancer.yaml) 참조.

## ExternalName
서비스 name 을 externalName 값 (=도메인) 과 매칭한다. 이 타입은 클러스터 내부에서 외부로 접근할 때 주로 사용한다 (외부 DB 서버 접속 등).
[예제](./externalname.yaml) 참조.
