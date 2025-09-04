**Делаем в minikube кластер из 5 нод**:

`minikube start --nodes=5 -p my-cluster`

**Распределяем ноды по трем зонам**:

```
kubectl label node my-cluster topology.kubernetes.io/zone=zone-a
kubectl label node my-cluster-m02 topology.kubernetes.io/zone=zone-a
kubectl label node my-cluster-m03 topology.kubernetes.io/zone=zone-b
kubectl label node my-cluster-m04 topology.kubernetes.io/zone=zone-b
kubectl label node my-cluster-m05 topology.kubernetes.io/zone=zone-c
kubectl get nodes --show-labels
```

**Деплоим приложение**:

`kubectl apply -f .\deployment.yml`

**Проверяем запуск подов**:

`kubectl get pods -o wide`

**Применяем настройки HorizontalPodAutoscaler**:

`kubectl apply -f .\hpa.yml`

**Авто-скейлинг под нагрузкой**.
Запускаем временный контейнер в поде:

```
kubectl get pods
kubectl debug -it pod/<один из подов> --image=ubuntu -- bash
```

Ставим в него утилиту stress и создаем нагрузку на CPU:

```
# apt update
# apt install -y stress
# stress --cpu 10 --timeout 60
```

Смотрим, как HPA увеличивает количество реплик:

`kubectl get hpa`

**Измененяем настройки HPA в зависимости от времени при помощи CronJob**. Для примера пусть ночью будет 2 реплики, а максимум 3

```
kubectl apply -f .\scaler-rbac.yml
kubectl apply -f .\cronjob_scale_day.yml
kubectl apply -f .\cronjob_scale_night.yml
```

Проверка cronjob'a:

```
create job --from=cronjob/hpa-night-adjust test-hpa-job
kubectl get jobs
kubectl get hpa
```

