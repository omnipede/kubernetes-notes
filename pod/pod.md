# Pod

```kubectl get pods -o wide``` 명령어로 pod 이 어떤 node 위에서 실행되고 있는지 확인할 수 있다.

## Pod 생성 방법

* 명령어로 만들기

```
$ kubectl run whoami --image subicura/whoami:1 
$ kubectl exec -it whoami sh
```

* 파일로 만들기

```
$ kubectl apply -f ./whoami-pod.yaml
$ kubectl delete -f ./whoami-pod.yaml
$ kubectl delete -f ./
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: whoami
  labels:
    type: app
spec:
  containers:
    - name: app
      image: subicura/whoami:1
```

## Pod 의 health check
* ```livenessProbe```: 프로세스가 살아있는지 확인

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: whoami-lp
  labels:
    type: app
spec:
  containers:
  - name: app
    image: subicura/whoami:1
    livenessProbe:
      httpGet:
        # 다음 endpoint 에 대해 주기적으로 요청을 보내서 프로세스가 살아있는지 확인한다
        path: /not/exist
        port: 8080
      initialDelaySeconds: 5
      timeoutSeconds: 2 # Default 1
      periodSeconds: 5 # Defaults 10
      failureThreshold: 1 # Defaults 3
```

* ```readinessProbe```: 프로세스가 준비되었는지 확인

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: whoami-rp
  labels:
    type: app
spec:
  containers:
  - name: app
    image: subicura/whoami:1
    readinessProbe:
      httpGet:
        path: /not/exist
        port: 8080
      initialDelaySeconds: 5
      timeoutSeconds: 2 # Default 1
      periodSeconds: 5 # Defaults 10
      failureThreshold: 1 # Defaults 3
```

* ```livenessProbe``` 와 ```readinessProbe``` 의 차이점  
  ```livenessProbe``` 는 pod 을 실행시킨 뒤 프로세스를 확인하고 ```readinessProbe``` 는 프로세스를 확인하고 pod 을 실행한다.
  describe 로 ```livenessProbe``` 예제 pod 과 ```readinessProbe``` 예제 pod 를 비교하면 차이를 알 수 있다.
```
NAME        READY   STATUS    RESTARTS   AGE
whoami-lp   1/1     Running   5          3m39s
whoami-rp   0/1     Running   0          3m43s
```
```livenessProbe``` 예제의 경우는 pod 이 ready 상태지만 livenessProbe 에 실패해 주기적으로 pod 이 재실행되고 있다. 반면 ```readinessProbe``` 예제는 readinessProbe 에 실패해서 pod 이 ready 상태가 아닌 것을 알 수 있다.

## Multi container pod 예제

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: whoami-redis
  labels:
    type: stack
spec:
  containers:
  - name: app
    image: subicura/whoami-redis:1
    env:
    - name: REDIS_HOST
      value: "localhost"
  - name: db
    image: redis
```

아래 명령어로 pod 내부의 컨테이너별로 로그를 확인할 수 있다.

```
$ kubectl logs whoami-redis app
$ kubectl logs whoami-redis db
```

***docker-compose 는 같은 네트워크 대역을 쓸 뿐, 컨테이너 별로 ip address 는 달랐다. 하지만 pod 내부의 컨테이너들은 동일한 ip 를 쓴다.***