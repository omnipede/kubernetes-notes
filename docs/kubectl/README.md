# Kubectl 주요 명령어

* Alias
```
$ alias k='kubectl'
```

* Get kube objects (nodes or pods or services or etc ...)
```
$ kubectl get all
$ kubectl get all --all-namespaces
$ kubectl get nodes
$ kubectl get nodes -o wide
$ kubectl get nodes -o yaml
$ kubectl get nodes -o json
```

* Exec container
```
$ kubectl exec my-pod -c my-container -- /bin/bash
```

* Describe kube objects (object 상태 확인)
```
# kubectl describe type/name
$ kubectl describe node/<node name>
```

* Delete kube objects

```
$ kubectl delete -f path/to/YOUR_KUBE_FILE.yaml
# kubectl delete type/object

$ kubectl delete pods --all
$ kubectl delete deployment --all
$ kubectl delete all --all -n $(NAME_SPACE)
```