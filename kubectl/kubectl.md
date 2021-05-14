# Kubectl 주요 명령어

* Alias
```
$ alias k='kubectl'
```

* Get kube objects (nodes or pods or services or etc ...)
```
$ kubectl get all
$ kubectl get nodes
$ kubectl get nodes -o wide
$ kubectl get nodes -o yaml
$ kubectl get nodes -o json
```

* Describe kube objects (object 상태 확인)
```
# kubectl describe type/name
$ kubectl describe node/<node name>
```
