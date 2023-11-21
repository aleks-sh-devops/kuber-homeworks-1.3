# Домашнее задание к занятию «Запуск приложений в K8S»

### Цель задания

В тестовой среде для работы с Kubernetes, установленной в предыдущем ДЗ, необходимо развернуть Deployment с приложением, состоящим из нескольких контейнеров, и масштабировать его.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) Init-контейнеров.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к репликам приложения из другого Pod

1. Создать Deployment приложения, состоящего из двух контейнеров — nginx и multitool. Решить возникшую ошибку.

Создаем пространство имен под ДЗ:
```
kubectl get ns
NAME              STATUS   AGE
kube-system       Active   7d15h
kube-public       Active   7d15h
kube-node-lease   Active   7d15h
default           Active   7d15h
lesson2           Active   14h
usrcon@cli-k8s-01:~/manifests/02_dz_kuber_1.3$ kubectl apply -f ~/manifests/02_dz_kuber_1.3/01_namespace.yml
namespace/dz3 created
usrcon@cli-k8s-01:~/manifests/02_dz_kuber_1.3$ kubectl get ns
NAME              STATUS   AGE
kube-system       Active   7d15h
kube-public       Active   7d15h
kube-node-lease   Active   7d15h
default           Active   7d15h
lesson2           Active   14h
dz3               Active   4s
```
Запускаем деплоймент с одной репликой:
```
kubectl apply -f ~/manifests/02_dz_kuber_1.3/02_deploy_nginx_multitool.yml
deployment.apps/dpl-nginx-multitool created
```
Смотрим ПОДы в созданном неймспейсе:
```
usrcon@cli-k8s-01:~/manifests/02_dz_kuber_1.3$ kubectl get pods -n dz3 -o wide
NAME                                   READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
dpl-nginx-multitool-645c8c8575-hgmwp   2/2     Running   0          9s    10.1.198.88   microk8s-01   <none>           <none>
```

Проверяем, что контейнеры в ПОДе отвечают:
```
usrcon@cli-k8s-01:~/manifests/02_dz_kuber_1.3$ kubectl exec -n dz3 dpl-nginx-multitool-645c8c8575-hgmwp -- curl localhost:31080
Defaulted container "nginx" out of: nginx, multitool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   155  100   155    0     0   7750      0 --:--:-- --:--:-- --:--:--  8157
WBITT Network MultiTool (with NGINX) - dpl-nginx-multitool-645c8c8575-hgmwp - 10.1.198.88 - HTTP: 31080 , HTTPS: 443 . (Formerly praqma/network-multitool)
usrcon@cli-k8s-01:~/manifests/02_dz_kuber_1.3$
usrcon@cli-k8s-01:~/manifests/02_dz_kuber_1.3$ kubectl exec -n dz3 dpl-nginx-multitool-645c8c8575-hgmwp -- curl localhost:80
Defaulted container "nginx" out of: nginx, multitool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0    99k      0 --:--:-- --:--:-- --:--:--  119k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
usrcon@cli-k8s-01:~/manifests/02_dz_kuber_1.3$ kubectl exec -n dz3 dpl-nginx-multitool-645c8c8575-hgmwp -- curl localhost:443
Defaulted container "nginx" out of: nginx, multitool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<html>
<head><title>400 The plain HTTP request was sent to HTTPS port</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<center>The plain HTTP request was sent to HTTPS port</center>
<hr><center>nginx/1.24.0</center>
</body>
</html>
100   255  100   255    0     0  63750      0 --:--:-- --:--:-- --:--:-- 63750
```

2. После запуска увеличить количество реплик работающего приложения до 2.
02_deploy_nginx_multitool.yml: replicas: 1 => replicas: 2
Запускаем деплоймент с двумя репликами:
```
kubectl apply -f ~/manifests/02_dz_kuber_1.3/02_deploy_nginx_multitool.yml
deployment.apps/dpl-nginx-multitool configured
```

Смотрим ПОДы в созданном неймспейсе:
```
usrcon@cli-k8s-01:~/manifests/02_dz_kuber_1.3$ kubectl get pods -n dz3 -o wide
NAME                                   READY   STATUS    RESTARTS   AGE    IP            NODE          NOMINATED NODE   READINESS GATES
dpl-nginx-multitool-645c8c8575-hgmwp   2/2     Running   0          11m    10.1.198.88   microk8s-01   <none>           <none>
dpl-nginx-multitool-645c8c8575-zp2t8   2/2     Running   0          103s   10.1.63.163   microk8s-02   <none>           <none>
```

Проверяем ответ второго ПОДа:
```
usrcon@cli-k8s-01:~/manifests/02_dz_kuber_1.3$ kubectl exec -n dz3 dpl-nginx-multitool-645c8c8575-zp2t8 -- curl localhost:31080
Defaulted container "nginx" out of: nginx, multitool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   155  100   155    0     0  22142      0 --:--:-- --:--:-- --:--:-- 22142
WBITT Network MultiTool (with NGINX) - dpl-nginx-multitool-645c8c8575-zp2t8 - 10.1.63.163 - HTTP: 31080 , HTTPS: 443 . (Formerly praqma/network-multitool)
usrcon@cli-k8s-01:~/manifests/02_dz_kuber_1.3$ kubectl exec -n dz3 dpl-nginx-multitool-645c8c8575-zp2t8 -- curl localhost:80
Defaulted container "nginx" out of: nginx, multitool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0  76500      0 --:--:-- --:--:-- --:--:-- 76500
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
usrcon@cli-k8s-01:~/manifests/02_dz_kuber_1.3$ kubectl exec -n dz3 dpl-nginx-multitool-645c8c8575-zp2t8 -- curl localhost:443
Defaulted container "nginx" out of: nginx, multitool
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<html>
<head><title>400 The plain HTTP request was sent to HTTPS port</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<center>The plain HTTP request was sent to HTTPS port</center>
<hr><center>nginx/1.24.0</center>
</body>
</html>
100   255  100   255    0     0  63750      0 --:--:-- --:--:-- --:--:-- 63750
```

3. Продемонстрировать количество подов до и после масштабирования.
Есть

4. Создать Service, который обеспечит доступ до реплик приложений из п.1.
5. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложений из п.1.




------

### Задание 2. Создать Deployment и обеспечить старт основного контейнера при выполнении условий

1. Создать Deployment приложения nginx и обеспечить старт контейнера только после того, как будет запущен сервис этого приложения.
2. Убедиться, что nginx не стартует. В качестве Init-контейнера взять busybox.
3. Создать и запустить Service. Убедиться, что Init запустился.
4. Продемонстрировать состояние пода до и после запуска сервиса.

------
