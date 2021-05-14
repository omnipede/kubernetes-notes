# ReplicaSet
레플리카셋은 실행되는 pod 개수에 대한 가용성을 보증 하며 pod 개수만큼 항상 실행될 수 있도록 관리.
레플리카셋은 실전에서는 많이 사용하지 않는다고 함. 실제로는 ```deployment``` 를 주로 사용한다.

## 기본 예제

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: whoami-rs
spec:
  # 유지할 pod 개수
  replicas: 1
  # Pod 의 개수를 유지할 때 다음 selector 를 기준으로 판단한다
  selector:
    matchLabels:
      type: app
      service: whoami
  # 실행할 pod
  template:
    metadata:
      # Selector 에서 선언한 label 을 metadata 에서 명시한다
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

```
$ kubectl get pods --show-labels # Pod 의 label 확인
$ kubectl label pod/whoami-rs-xxxx service- # Service label 해제
$ kubectl label pod/whoami-rs-xxxx service=whoami # Service label 추가
```

만약 실행중인 pod 의 label 을 해제하면 label 을 가진 replicaset 이 새로운 pod 을 실행시키는 것을 알 수 있다. 
ReplicaSet 이 항상 일정한 개수의 pod 을 유지시키기 때문이다.

```
NAME              READY   STATUS    RESTARTS   AGE     LABELS
whoami-rs-ncxgl   1/1     Running   0          7s      service=whoami,type=app
whoami-rs-qnvhs   1/1     Running   0          7m15s   type=app
```

## 스케일 아웃 예제

* 명령어로 실행

```
$ kubectl scale --replicas=3 -f whoami-rs.yaml
```

* 파일로 실행

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: whoami-rs
spec:
  replicas: 4
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

```
$ kubectl apply -f whoami-rs-scaled.yaml
```