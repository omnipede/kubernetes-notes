# Istio visualization

Istio 로 구성된 service mesh 아키텍처를 시각화하는 방법을 정리. [쿠버네티스 공식 문서](https://istio.io/latest/docs/tasks/observability/kiali/) 참조.

![image](https://user-images.githubusercontent.com/41066039/119782878-a478f980-bf07-11eb-9f5a-42d462920754.png)


## 설치 방법

### ```Kiali``` 설치

```
$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.10/samples/addons/kiali.yaml
$ kubectl -n istio-system get svc kiali
```

### ```Prometheus``` 설치

```
$ kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.10/samples/addons/prometheus.yaml
```

## 실행 방법

```istioctl``` 을 사용해서 dashboard 실행

```
$ istioctl dashboard kiali
```

이후 ```http://localhost:20001/kiali``` 로 접속하면 대시보드를 확인할 수 있다. 
앞서 살펴보았던 [istio bookinfo 예제](../README.md) 의 그래프를 대시보드상에서 확인해보자.

![image](https://user-images.githubusercontent.com/41066039/119784280-038b3e00-bf09-11eb-8d90-afa83d5c7b59.png)
