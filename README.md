# Домашнее задание Сартисона Евгения N4 #
Цель:
развернуть отказоустойчивый кластер Couchbase, наполнить его тестовыми данными и проверить поведение системы при сбоях узлов.



## Сама работа #

Выполнил самую базовую установку, с остальными пунктами не успеваю по времени. Документация закрытая, не так просто выполнить настройку.

Через VirtualBox поднял Minikube и создал CouchBase кластер через [Create a Couchbase cluster using Kubernetes](https://kubernetes.io/blog/2016/08/create-couchbase-cluster-using-kubernetes/)

Создать  master Replication Controller
```
root@course:~/couchbase/couchbase-kubernetes-master/cluster# cat cluster-master.yml 
apiVersion: v1
kind: ReplicationController
metadata:
  name: couchbase-master-rc
spec:
  replicas: 1
  selector:
    app: couchbase-master-pod
  template:
    metadata:
      labels:
        app: couchbase-master-pod
    spec:
      containers:
      - name: couchbase-master
        image: arungupta/couchbase:k8s
        env:
          - name: TYPE
            value: MASTER
        ports:
        - containerPort: 8091
---
apiVersion: v1
kind: Service
metadata: 
  name: couchbase-master-service
  labels: 
    app: couchbase-master-service
spec: 
  ports:
    - port: 8091
  selector: 
    app: couchbase-master-pod
  type: LoadBalancer
root@course:~/couchbase/couchbase-kubernetes-master/cluster# kubectl create -f cluster-master.yml   
replicationcontroller/couchbase-master-rc created
service/couchbase-master-service created
```


Проверка сервисов
```
root@course:~/couchbase/couchbase-kubernetes-master/cluster# kubectl get svc
NAME                       TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
couchbase-master-service   LoadBalancer   10.98.96.45   <pending>     8091:31725/TCP   95s
kubernetes                 ClusterIP      10.96.0.1     <none>        443/TCP          20h

root@course:~/couchbase/couchbase-kubernetes-master/cluster# kubectl get po  
NAME                        READY   STATUS    RESTARTS   AGE
couchbase-master-rc-tgtww   1/1     Running   0          4m37s

root@course:~/couchbase/couchbase-kubernetes-master/cluster# kubectl get rc  
NAME                  DESIRED   CURRENT   READY   AGE
couchbase-master-rc   1         1         1       5m17s
```



Создатт рабочий Replication Controller
```
root@course:~/couchbase/couchbase-kubernetes-master/cluster# cat cluster-worker.yml 
apiVersion: v1
kind: ReplicationController
metadata:
  name: couchbase-worker-rc
spec:
  replicas: 1
  selector:
    app: couchbase-worker-pod
  template:
    metadata:
      labels:
        app: couchbase-worker-pod
    spec:
      containers:
      - name: couchbase-worker
        image: arungupta/couchbase:k8s
        env:
          - name: TYPE
            value: "WORKER"
          - name: COUCHBASE_MASTER
            value: "couchbase-master-service"
          - name: AUTO_REBALANCE
            value: "false"
        ports:
        - containerPort: 8091
root@course:~/couchbase/couchbase-kubernetes-master/cluster# kubectl create -f cluster-worker.yml   
replicationcontroller/couchbase-worker-rc created
```

Проверить сервисы
```
root@course:~/couchbase/couchbase-kubernetes-master/cluster# kubectl get rc 
NAME                  DESIRED   CURRENT   READY   AGE
couchbase-master-rc   1         1         1       7m44s
couchbase-worker-rc   1         1         1       36s
root@course:~/couchbase/couchbase-kubernetes-master/cluster# kubectl get po  
NAME                        READY   STATUS    RESTARTS   AGE
couchbase-master-rc-tgtww   1/1     Running   0          8m
couchbase-worker-rc-ll6nj   1/1     Running   0          53s
```

Добавить ноды в кластер
```
root@course:~/couchbase/couchbase-kubernetes-master/cluster# kubectl scale rc couchbase-worker-rc --replicas=3 
replicationcontroller/couchbase-worker-rc scaled
```

Проверка сервисов
```
  root@course:~/couchbase/couchbase-kubernetes-master/cluster# kubectl get rc 
NAME                  DESIRED   CURRENT   READY   AGE
couchbase-master-rc   1         1         1       9m43s
couchbase-worker-rc   3         3         3       2m35s
root@course:~/couchbase/couchbase-kubernetes-master/cluster# kubectl get po  
NAME                        READY   STATUS    RESTARTS   AGE
couchbase-master-rc-tgtww   1/1     Running   0          9m51s
couchbase-worker-rc-bzbfm   1/1     Running   0          61s
couchbase-worker-rc-ll6nj   1/1     Running   0          2m44s
couchbase-worker-rc-x4bj2   1/1     Running   0          61s
```

Базовую настройку выполнил.



  
