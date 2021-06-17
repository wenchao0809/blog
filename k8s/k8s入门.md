### 查看容器日志

~~~sh
k logs -f --tail=1000  advice-backend-55648c48b4-cktrc -c advice-backend
~~~

### 查看pod

~~~sh
k get pods
k get pods -o wide
~~~

### 进入容器

~~~sh
 kubectl exec -it advice-backend-55648c48b4-cktrc -c advice-backend /bin/sh
~~~

### 删除pod

~~~sh
kubectl get deployment
kubectl delete deployment advice-backend
~~~