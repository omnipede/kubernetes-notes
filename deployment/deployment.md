# Deployment

ReplicaSet 과 거의 유사하다.
버저닝, 롤백 등의 기능이 추가된 ReplicaSet. ReplicaSet 의 기능을 Deployment 가 이용한다고 보면 됨

## 기본 예제

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      type: app
      service: whoami
  template:
    metadata:
      labels:
        type: app
        service: whoami
    spec:
      containers:
      - name: whoami
        image: subicura/whoami:1
        livenessProbe:
          httpGet:
            path: /
            port: 4567
```

위 예제에서 image 를 ```subicura/whoami:2``` 로 수정하면 기존에 실행중이던 pod 3개가 내려가고
수정된 image 로 pod 3개가 다시 실행된다.

```
NAME                             READY   STATUS        RESTARTS   AGE
whoami-deploy-698f4ffd64-b78w8   1/1     Terminating   0          73s
whoami-deploy-698f4ffd64-qz8fp   1/1     Terminating   0          73s
whoami-deploy-698f4ffd64-zmrl9   1/1     Terminating   0          73s
whoami-deploy-8686bdc7cc-hcdjf   1/1     Running       0          6s
whoami-deploy-8686bdc7cc-jljqg   1/1     Running       0          5s
whoami-deploy-8686bdc7cc-rpg5b   1/1     Running       0          14s
```

그리고 deployment 의 history 를 아래 명령어로 살펴보면 revision 2개를 확인할 수 있다.

```
$ kubectl rollout history deployment/whoami-deploy

REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

각각의 revision 의 상세를 보고 싶으면 다음 명령어를 사용한다.

```
$ kubectl rollout history deployment/whoami-deploy --revision=2

Pod Template:
  Labels:       pod-template-hash=8686bdc7cc
        service=whoami
        type=app
  Containers:
   whoami:
    Image:      subicura/whoami:2
    Port:       <none>
    Host Port:  <none>
    Liveness:   http-get http://:4567/ delay=0s timeout=1s period=10s #success=1 #failure=3
    Environment:        <none>
    Mounts:     <none>
  Volumes:      <none>
```

만약 현재 revision 대신 이전 revision 을 다시 배포하고 싶으면 다음 명령어를 사용한다.

```
$ kubectl rollout undo deployment/whoami-deploy
$ kubectl rolloiut history deployment/whoami-deploy

REVISION  CHANGE-CAUSE
2         <none>
3         <none>
```