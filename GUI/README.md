# GUI
쿠버네티스 GUI 대시보드 관련 내용 정리

[쿠버네티스 공식 문서](https://kubernetes.io/ko/docs/tasks/access-application-cluster/web-ui-dashboard/) 를 참조함

## 배포 방법
다음 커맨드를 실행한다.

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```

이 커맨드는 ```kubernetes-dashboard``` namespace 에 dashboard 관련 리소스를 생성해준다.
배포 이후 ```kubectl proxy``` 명령어로 dashboard 리소스에 대해 접근할 수 있게 한다. Proxy 중인 상태에서 

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

로 접속하면 된다.

본 문서에서는 ```kubectl proxy``` 를 이용해서 대시보드에 접근하고 있지만 실제 운영환경에서는 ```kubernetes-dashboard``` namespace 에 ingress 나 NodePort 리소스를 생성해서 dashboard 에 
접근하는게 좋을 것 같다.

## 유저 생성

### ServiceAccount 리소스 생성

```kubernetes-dashboard``` namespace 에 [Service Account](./user-service-account.yaml) 를 생성한다.
```
$ kubectl apply -f ./user-service-account.yaml
```

### ClusterRoleBinding 리소스 생성

생성한 Service Account 에 Cluster Role 을 binding 한다.

```
$ kubectl apply -f ./user-cluster-role-binding.yaml
```

### Bearer 토큰 출력

```
$ kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```

출력된 토큰을 대시보드에 입력하면 로그인할 수 있다.

![image](https://user-images.githubusercontent.com/41066039/119760571-36233f80-bee5-11eb-9449-0df899b689b2.png)

