# Ingress

Domain, path 를 이용해서 네트워크와 다른 service object 를 연결해주는 리소스.

쿠버네티스의 서비스 리소스는 L4 레이어로 TCP 단에서 pod 들을 밸런싱한다. 따라서 서비스는 도메인명 또는 URL path 에 대해 라우팅이 불가능하다. 이러한 L7 로드밸런싱 기능을 제공하는
리소스를 ingress 라고 한다.

Ingress 자체는 클러스터 외부에서 접근해야 할 URL 을 사용할 수 있도록 하고, 로드 밸런싱, SSL 인증서 처리, 도메인 기반 가상 호스팅 등의 규칙을 정의해둔 자원이고,
실제 동작하는 것은 ingress controller 다. 따라서 Ingress 를 사용하기 위해서는 먼저 ```ingress controller``` 를 설치해야 한다.

아래 명령어로 ingress controller 의 일종인 nginx-controller 를 설치할 수 있다.

```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/baremetal/deploy.yaml
```

설치 후 ```ingress-nginx``` namespace 상에 생성된 리소스는 다음과 같다.

```
$ k get all -n ingress-nginx

NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-zmndv        0/1     Completed   0          15h
pod/ingress-nginx-admission-patch-5rbmc         0/1     Completed   0          15h
pod/ingress-nginx-controller-5dbd9649d4-mvqkr   1/1     Running     1          15h

NAME                                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             NodePort    10.105.83.52     <none>        80:31647/TCP,443:32696/TCP   15h
service/ingress-nginx-controller-admission   ClusterIP   10.100.203.171   <none>        443/TCP                      15h

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           15h
```

그리고 브라우저로 ```ingress-nginx-controller``` service 의 포트에 접속하면 아직 ingress 에 k8s 서비스가 연동되어 있지 않아서 ```Not found```에러 페이지가 뜬다.
![image](https://user-images.githubusercontent.com/41066039/119438647-a69e5500-bd5b-11eb-9675-592cc0f601d0.png)

[예제](./ingress-v1.yaml) 를 실행시키면 ingress 에 k8s 서비스를 연동해볼 수 있다.
단, 예제 실행 전, ingress 에 domain name 을 붙여야한다. [ngrok](https://ngrok.com/) 라는 툴을 이용하면 간단하게 도메인을 생성할 수 있다.
```ngrok``` 를 이용해서 도메인 생성 시 도메인이 ```ingress-controller``` 의 포트로 포워딩하도록 한다. 현재 생성된 ingress controller 의 포트가 31647 이므로 31647 로 포워딩하자.
```
$ ngrok http 31647
```
포트포워딩 설정 후 ngrok 로 생성된 도메인을 [예제](./ingress-v1.yaml) 에 넣어주자.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: whoami-v1
  annotations:
    ingress.kubernetes.io/rewrite-target: "/"
    ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
    # 아래 도메인을 수정.
    - host: "c0b77ef334e6.ngrok.io"
      ...
```
예제에 도메인을 넣은 뒤 [예제](./ingress-v1.yaml) 를 실행시키고 브라우저를 통해 도메인에 접속하면 쿠버네티스 서비스를 확인할 수 있다.
![image](https://user-images.githubusercontent.com/41066039/119450225-8f1c9780-bd6e-11eb-8913-812bc7dcda0e.png)