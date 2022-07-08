## Pod

Создаем namespace

```
kubectl create ns pod-demo
```

Смотрим на поведение pod с restartPolicy = Always, который корректно завершает работу:  
```
args: [ "sleep 5; ls -lh;" ]
restartPolicy: Always
```

Применяем манифест  
```
kubectl apply -f pod-example.yaml
```
Смотрим состояние pod
```
kubectl get po -A -w
```

Удаляем pod
```
kubectl delete -f pod-example.yaml
```

Смотрим на поведение pod с restartPolicy = Always, который не корректно завершает работу:  

```
args: [ "sleep 5; lss -lh;" ]
restartPolicy: Always
```

```
kubectl apply -f pod-example.yaml
kubectl get po -A -w
kubectl delete -f pod-example.yaml
```

Смотрим на поведение pod с restartPolicy = OnFailure, который корректно завершает работу:  

```
args: [ "sleep 5; ls -lh;" ]
restartPolicy: OnFailure
```

```
kubectl apply -f pod-example.yaml
kubectl get po -A -w
kubectl delete -f pod-example.yaml
```

Смотрим на поведение pod с restartPolicy = OnFailure, который не корректно завершает работу:  

```
args: [ "sleep 5; lss -lh;" ]
restartPolicy: OnFailure
```

```
kubectl apply -f pod-example.yaml
kubectl get po -A -w
kubectl delete -f pod-example.yaml
```


Смотрим на поведение pod с restartPolicy = Never, который корректно завершает работу:  

```
args: [ "sleep 5; ls -lh;" ]
restartPolicy: Never
```

```
kubectl apply -f pod-example.yaml
kubectl get po -A -w
kubectl delete -f pod-example.yaml
```

Смотрим на поведение pod с restartPolicy = Never, который не корректно завершает работу:  

```
args: [ "sleep 5; lss -lh;" ]
restartPolicy: Never
```

```
kubectl apply -f pod-example.yaml
kubectl get po -A -w
kubectl delete -f pod-example.yaml
```

## Replicaset

Создаем namespace

```
kubectl create ns replicaset-demo
```

Применяем манифест  
```
kubectl apply -f replicaset-demo.yaml
```
Смотрим состояние pod
```
kubectl get po -A -w
```
Смотрим состояние replicaset
```
kubectl get rs -n replicaset-demo
```
Скалируем реплики в большую сторону и смотрим на их состояние
```
kubectl scale --replicas=5 -n replicaset-demo replicaset/replicaset-demo
kubectl get rs -n replicaset-demo
kubectl get po -n replicaset-demo
```
Скалируем реплики в меньшую сторону и смотрим на их состояние
```
kubectl scale --replicas=1 -n replicaset-demo replicaset/replicaset-demo
kubectl get rs -n replicaset-demo
kubectl get po -n replicaset-demo
```
Смотрим версии Images в pod этой rs
```
kubectl describe po -n replicaset-demo | grep Image:
```
После чего меняем в манифесте версию, применяем его, смотрим состояние pod и Image
```
kubectl apply -f replicaset-demo.yaml
kubectl describe po -n replicaset-demo | grep Image:
```
Удаляем rs
```
kubectl delete -f replicaset-demo.yaml

```

## Deployment
Создаем namespace

```
kubectl create ns deployment-demo
```
Применяем манифест  
```
kubectl apply -f deployment-demo.yaml
```
Смотрим состояние pod
```
kubectl get po -A -w
```
Смотрим состояние deployment
```
kubectl get deployment -n deployment-demo
```
Смотрим состояние rs в этом дпелойменте
```
kubectl get rs -n deployment-demo
```
Меняем в манифесте версию Image, применяем его, смотрим состояние pod, rs, deployment и Image 
```
kubectl get po -n deployment-demo -w
kubectl get rs -n deployment-demo
kubectl describe po -n deployment-demo | grep Image:
```
Скалируем deployment в большую сторону и смотрим на его состояние
```
kubectl scale --replicas=7 -n deployment-demo deployment/deployment-demo
kubectl get rs -n deployment-demo
kubectl get po -n deployment-demo
```
Скалируем реплики в меньшую сторону и смотрим на их состояние
```
kubectl scale --replicas=3 -n deployment-demo deployment/deployment-demo
kubectl get rs -n deployment-demo
kubectl get po -n deployment-demo
```
Удаляем deployment
```
kubectl delete -f deployment-demo.yaml
```
## DaemonSet

Создаем namespace

```
kubectl create ns daemonset-demo
```
Применяем манифест  
```
kubectl apply -f daemonset-demo.yaml
```
Смотрим состояние pod
```
kubectl get po -n
daemonset-demo -o wide
```
Смотрим состояние daemonset
```
kubectl get daemonset -n daemonset-demo
```

Меняем в манифесте версию Image, применяем его, смотрим состояние pod, daemonset и Image 
```
kubectl apply -f daemonset-demo.yaml
kubectl get po -n daemonset-demo -w
kubectl describe po -n daemonset-demo | grep Image:
```

## StatefulSet

## Job

## CronJob
