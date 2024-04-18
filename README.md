# raacharg_platform
raacharg Platform repository

=======
## ДЗ №1.

### Последовательность выполнения ДЗ.
1. Настоил окружение.
2. Написал `Dockerfile`, собрал контейнер загрузил на https://hub.docker.com/.
3. Написал манифест `web-pod.yaml`, запустил соответствующий под.

### Решение заданий

`Разберитесь почему все pod в namespace kube-system восстановились
после удаления. Укажите причину в описании PR`

Ответ: kube-proxy управляется Daemon-set, core-dns управляется ReplicaSet, остальное пересоздается kubelet, который в свою очередь поднимается systemd.

`Hipster Shop | Задание со ⭐`

Ответ: Обратил внимание на то, что сервис не может найти переменные. Добавил переменные из подсказки в манифест. Запустил -все работает.

=======
## ДЗ №2.
1. Поднял kind кластер
```
kubectl get nodes
NAME                  STATUS   ROLES           AGE   VERSION
kind-control-plane    Ready    control-plane   88m   v1.29.2
kind-control-plane2   Ready    control-plane   88m   v1.29.2
kind-control-plane3   Ready    control-plane   87m   v1.29.2
kind-worker           Ready    <none>          87m   v1.29.2
kind-worker2          Ready    <none>          87m   v1.29.2
kind-worker3          Ready    <none>          87m   v1.29.2
```

2. Запустил ReplicaSet из задания и получил ошибку
```
kubectl apply -f frontend-replicaset.yaml 
Error from server (BadRequest): error when creating "frontend-replicaset.yaml": ReplicaSet in version "v1" cannot be handled as a ReplicaSet: strict decoding error: unknown field "spec.spec"
```
3. Исправил ReplicaSet.
```
kubectl get pods -l app=frontend
NAME             READY   STATUS    RESTARTS   AGE
frontend-sds2l   1/1     Running   0          47s
```
4. Увеличил количество реплик.
```
kubectl scale replicaset frontend --replicas=3
replicaset.apps/frontend scaled

kubectl get pods -l app=frontend
NAME             READY   STATUS    RESTARTS   AGE
frontend-82cqs   1/1     Running   0          6m25s
frontend-grrzn   1/1     Running   0          6m25s
frontend-sds2l   1/1     Running   0          7m35s
```
5. Убедился, что контроллер управляет тремя репликами и проверил восстановление подов после удаления.
```
kubectl get rs frontend
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       8m24s

kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w
NAME             READY   STATUS    RESTARTS   AGE
frontend-82cqs   1/1     Running   0          9m1s
frontend-grrzn   1/1     Running   0          9m1s
frontend-sds2l   1/1     Running   0          10m
frontend-82cqs   1/1     Terminating   0          9m1s
frontend-grrzn   1/1     Terminating   0          9m1s
frontend-qx88p   0/1     Pending       0          0s
frontend-qx88p   0/1     Pending       0          0s
frontend-sds2l   1/1     Terminating   0          10m
frontend-7wf4v   0/1     Pending       0          0s
frontend-7wf4v   0/1     Pending       0          0s
frontend-qx88p   0/1     ContainerCreating   0          0s
frontend-txglv   0/1     Pending             0          0s
frontend-7wf4v   0/1     ContainerCreating   0          0s
frontend-txglv   0/1     Pending             0          0s
frontend-txglv   0/1     ContainerCreating   0          0s
frontend-82cqs   0/1     Terminating         0          9m1s
frontend-grrzn   0/1     Terminating         0          9m1s
frontend-sds2l   0/1     Terminating         0          10m
frontend-82cqs   0/1     Terminating         0          9m1s
frontend-82cqs   0/1     Terminating         0          9m1s
frontend-grrzn   0/1     Terminating         0          9m1s
frontend-82cqs   0/1     Terminating         0          9m1s
frontend-sds2l   0/1     Terminating         0          10m
frontend-grrzn   0/1     Terminating         0          9m1s
frontend-grrzn   0/1     Terminating         0          9m1s
frontend-sds2l   0/1     Terminating         0          10m
frontend-sds2l   0/1     Terminating         0          10m
frontend-7wf4v   1/1     Running             0          1s
frontend-qx88p   1/1     Running             0          1s
frontend-txglv   1/1     Running             0          1s
```
6. Повторно применил манифест. Количество реплик уменьшилось до одной.
```
kubectl get pods -l app=frontend
NAME             READY   STATUS    RESTARTS   AGE
frontend-txglv   1/1     Running   0          60s
```
7. Изменил манифест, чтобы разворачивалось сразу три реплики.
8. Перетегировал образ.
```
kubectl apply -f frontend-replicaset.yaml 
replicaset.apps/frontend configured
kubectl get pods -l app=frontend
NAME             READY   STATUS    RESTARTS   AGE
frontend-gfv5l   1/1     Running   0          5s
frontend-txglv   1/1     Running   0          2m51s
frontend-x45lc   1/1     Running   0          5s
```
9. Проверил образ указанный в ReplicaSet и образ из которого запущены pods
```
kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'
raacharg/hipster-frontend:v0.0.2
```
```
kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'
raacharg/hipster-frontend:v0.0.2 raacharg/hipster-frontend:v0.0.1
# Понимаю, что должен был увидеть только первую версию, но методичка не говорит о применении манифеста 
# при изменении количества реплик в манифесте на 3
```
10. Удалите все запущенные pod и после их пересоздания еще раз проверьте, из какого образа они развернулись.
```
kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'
raacharg/hipster-frontend:v0.0.2

kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'
raacharg/hipster-frontend:v0.0.2 raacharg/hipster-frontend:v0.0.2 raacharg/hipster-frontend:v0.0.2
```
11. Руководствуясь материалами лекции опишите произошедшую ситуацию, почему обновление ReplicaSet не повлекло обновление запущенных pod?
 - У меня обновилось, как и должно было. Но в случае, если у меня применен манифест с образом версии 1 и я обновляю в нем образ до версии 2 и применяю манифест снова, ReplicaSet не обновит образ, потому что контроллер Replicaset гарантирует только работу определенного количества реплик пода и не пересоздает и не обновляет существующие поды.

12. Скопируйте содержимое файла paymentservice-replicaset.yaml в файл paymentservice-deployment.yaml. Измените поле kind с ReplicaSet на Deployment. Применить манифест.
```
kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
frontend-7d4dbf654f-5qzdr   1/1     Running   0          4s
frontend-7d4dbf654f-dqfw9   1/1     Running   0          4s
frontend-7d4dbf654f-p8nfz   1/1     Running   0          4s

kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
frontend   3/3     3            3           12s

kubectl get rs
NAME                  DESIRED   CURRENT   READY   AGE
frontend-7d4dbf654f   3         3         3       98s
```
13. Обновил версию в образа в paymentservice-deployment.yaml.
```
kubectl apply -f paymentservice-deployment.yaml | kubectl get pods -l app=paymentservice -w
NAME                        READY   STATUS    RESTARTS   AGE
frontend-7d4dbf654f-5qzdr   1/1     Running   0          3m3s
frontend-7d4dbf654f-dqfw9   1/1     Running   0          3m3s
frontend-7d4dbf654f-p8nfz   1/1     Running   0          3m3s
frontend-85c48684d-gpt54    0/1     Pending   0          0s
frontend-85c48684d-gpt54    0/1     Pending   0          0s
frontend-85c48684d-gpt54    0/1     ContainerCreating   0          0s
frontend-85c48684d-gpt54    1/1     Running             0          1s
frontend-7d4dbf654f-dqfw9   1/1     Terminating         0          3m4s
frontend-85c48684d-l97st    0/1     Pending             0          0s
frontend-85c48684d-l97st    0/1     Pending             0          0s
frontend-85c48684d-l97st    0/1     ContainerCreating   0          0s
frontend-7d4dbf654f-dqfw9   0/1     Terminating         0          3m5s
frontend-7d4dbf654f-dqfw9   0/1     Terminating         0          3m5s
frontend-7d4dbf654f-dqfw9   0/1     Terminating         0          3m5s
frontend-7d4dbf654f-dqfw9   0/1     Terminating         0          3m5s
frontend-85c48684d-l97st    1/1     Running             0          2s
frontend-7d4dbf654f-p8nfz   1/1     Terminating         0          3m6s
frontend-85c48684d-rj4l5    0/1     Pending             0          0s
frontend-85c48684d-rj4l5    0/1     Pending             0          0s
frontend-85c48684d-rj4l5    0/1     ContainerCreating   0          0s
frontend-7d4dbf654f-p8nfz   0/1     Terminating         0          3m7s
frontend-7d4dbf654f-p8nfz   0/1     Terminating         0          3m7s
frontend-7d4dbf654f-p8nfz   0/1     Terminating         0          3m7s
frontend-7d4dbf654f-p8nfz   0/1     Terminating         0          3m7s
frontend-85c48684d-rj4l5    1/1     Running             0          2s
frontend-7d4dbf654f-5qzdr   1/1     Terminating         0          3m8s
frontend-7d4dbf654f-5qzdr   0/1     Terminating         0          3m9s
frontend-7d4dbf654f-5qzdr   0/1     Terminating         0          3m9s
frontend-7d4dbf654f-5qzdr   0/1     Terminating         0          3m9s
frontend-7d4dbf654f-5qzdr   0/1     Terminating         0          3m9s
```
14. Уедился, что все новые pod развернуты из образа второй версии и так же создано два rs. Так же проверил историю версий Deployment.
```
kubectl get deploy  frontend  -o=jsonpath='{.spec.template.spec.containers[0].image}'
raacharg/hipster-frontend:v0.0.2

kubectl get rs
NAME                  DESIRED   CURRENT   READY   AGE
frontend-7d4dbf654f   0         0         0       5m31s
frontend-85c48684d    3         3         3       2m28s

kubectrollout history deployment frontend
deployment.apps/frontend 
REVISION  CHANGE-CAUSE
3         <none>
4         <none>
```
15. Выполнил rollback.
```
ubectl rollout undo deployment frontend --to-revision=3 | kubectl get rs -l app=paymentservice -w
NAME                  DESIRED   CURRENT   READY   AGE
frontend-7d4dbf654f   0         0         0       10m
frontend-85c48684d    3         3         3       7m55s
frontend-7d4dbf654f   0         0         0       10m
frontend-7d4dbf654f   1         0         0       10m
frontend-7d4dbf654f   1         0         0       10m
frontend-7d4dbf654f   1         1         0       10m
frontend-7d4dbf654f   1         1         1       11m
frontend-85c48684d    2         3         3       7m57s
frontend-7d4dbf654f   2         1         1       11m
frontend-85c48684d    2         3         3       7m57s
frontend-7d4dbf654f   2         1         1       11m
frontend-85c48684d    2         2         2       7m57s
frontend-7d4dbf654f   2         2         1       11m
frontend-7d4dbf654f   2         2         2       11m
frontend-85c48684d    1         2         2       7m58s
frontend-85c48684d    1         2         2       7m58s
frontend-7d4dbf654f   3         2         2       11m
frontend-7d4dbf654f   3         2         2       11m
frontend-7d4dbf654f   3         3         2       11m
frontend-85c48684d    1         1         1       7m58s
frontend-7d4dbf654f   3         3         3       11m
frontend-85c48684d    0         1         1       8m
frontend-85c48684d    0         1         1       8m
frontend-85c48684d    0         0         0       8m
```
16. Deployment | Задание со звезочкой.
- Reverse Rolling Update
```
kubectl apply -f paymentservice-deployment-reverse.yaml | kubectl get pods -w
NAME                       READY   STATUS    RESTARTS   AGE
frontend-85c48684d-snzpt   1/1     Running   0          34s
frontend-85c48684d-v9dld   1/1     Running   0          34s
frontend-85c48684d-wx2c9   1/1     Running   0          34s
frontend-7d4dbf654f-4676v   0/1     Pending   0          0s
frontend-7d4dbf654f-4676v   0/1     Pending   0          0s
frontend-7d4dbf654f-4676v   0/1     ContainerCreating   0          0s
frontend-7d4dbf654f-4676v   1/1     Running             0          2s
frontend-85c48684d-snzpt    1/1     Terminating         0          36s
frontend-7d4dbf654f-xsqhw   0/1     Pending             0          0s
frontend-7d4dbf654f-xsqhw   0/1     Pending             0          0s
frontend-7d4dbf654f-xsqhw   0/1     ContainerCreating   0          0s
frontend-85c48684d-snzpt    0/1     Terminating         0          36s
frontend-85c48684d-snzpt    0/1     Terminating         0          37s
frontend-85c48684d-snzpt    0/1     Terminating         0          37s
frontend-85c48684d-snzpt    0/1     Terminating         0          37s
frontend-7d4dbf654f-xsqhw   1/1     Running             0          2s
frontend-85c48684d-wx2c9    1/1     Terminating         0          38s
frontend-7d4dbf654f-ftngz   0/1     Pending             0          0s
frontend-7d4dbf654f-ftngz   0/1     Pending             0          0s
frontend-7d4dbf654f-ftngz   0/1     ContainerCreating   0          0s
frontend-85c48684d-wx2c9    0/1     Terminating         0          38s
frontend-85c48684d-wx2c9    0/1     Terminating         0          39s
frontend-85c48684d-wx2c9    0/1     Terminating         0          39s
frontend-7d4dbf654f-ftngz   1/1     Running             0          2s
frontend-85c48684d-v9dld    1/1     Terminating         0          40s
frontend-85c48684d-v9dld    0/1     Terminating         0          40s
frontend-85c48684d-v9dld    0/1     Terminating         0          41s
frontend-85c48684d-v9dld    0/1     Terminating         0          41s
frontend-85c48684d-v9dld    0/1     Terminating         0          41s
```
- Аналог blue-green
```
kubectl apply -f paymentservice-deployment-bg.yaml | kubectl get pods -w
NAME                        READY   STATUS    RESTARTS   AGE
frontend-7d4dbf654f-86nxf   1/1     Running   0          20s
frontend-7d4dbf654f-cw5sw   1/1     Running   0          20s
frontend-7d4dbf654f-l4hhb   1/1     Running   0          20s
frontend-85c48684d-4qprs    0/1     Pending   0          0s
frontend-85c48684d-p6r4t    0/1     Pending   0          0s
frontend-85c48684d-4qprs    0/1     Pending   0          0s
frontend-85c48684d-5jq4g    0/1     Pending   0          0s
frontend-85c48684d-p6r4t    0/1     Pending   0          0s
frontend-85c48684d-5jq4g    0/1     Pending   0          0s
frontend-85c48684d-4qprs    0/1     ContainerCreating   0          0s
frontend-85c48684d-p6r4t    0/1     ContainerCreating   0          0s
frontend-85c48684d-5jq4g    0/1     ContainerCreating   0          0s
frontend-85c48684d-5jq4g    1/1     Running             0          1s
frontend-85c48684d-p6r4t    1/1     Running             0          1s
frontend-85c48684d-4qprs    1/1     Running             0          1s
frontend-7d4dbf654f-l4hhb   1/1     Terminating         0          21s
frontend-7d4dbf654f-cw5sw   1/1     Terminating         0          21s
frontend-7d4dbf654f-86nxf   1/1     Terminating         0          21s
frontend-7d4dbf654f-l4hhb   0/1     Terminating         0          21s
frontend-7d4dbf654f-cw5sw   0/1     Terminating         0          21s
frontend-7d4dbf654f-86nxf   0/1     Terminating         0          22s
frontend-7d4dbf654f-l4hhb   0/1     Terminating         0          22s
frontend-7d4dbf654f-86nxf   0/1     Terminating         0          22s
frontend-7d4dbf654f-cw5sw   0/1     Terminating         0          22s
frontend-7d4dbf654f-86nxf   0/1     Terminating         0          22s
frontend-7d4dbf654f-l4hhb   0/1     Terminating         0          22s
frontend-7d4dbf654f-cw5sw   0/1     Terminating         0          22s
frontend-7d4dbf654f-l4hhb   0/1     Terminating         0          22s
frontend-7d4dbf654f-86nxf   0/1     Terminating         0          22s
frontend-7d4dbf654f-cw5sw   0/1     Terminating         0          22s
```

17. Probes
```
kubectl apply -f frontend-deployment.yaml | kubectl get pods -w
NAME                       READY   STATUS    RESTARTS   AGE
frontend-c8699877c-9cxk2   0/1     Pending   0          0s
frontend-c8699877c-hfkzh   0/1     Pending   0          0s
frontend-c8699877c-9cxk2   0/1     Pending   0          1s
frontend-c8699877c-sbv6p   0/1     Pending   0          0s
frontend-c8699877c-hfkzh   0/1     Pending   0          0s
frontend-c8699877c-sbv6p   0/1     Pending   0          0s
frontend-c8699877c-9cxk2   0/1     ContainerCreating   0          1s
frontend-c8699877c-hfkzh   0/1     ContainerCreating   0          0s
frontend-c8699877c-sbv6p   0/1     ContainerCreating   0          0s
frontend-c8699877c-sbv6p   0/1     Running             0          1s
frontend-c8699877c-9cxk2   0/1     Running             0          2s
frontend-c8699877c-hfkzh   0/1     Running             0          1s
```
- Заменил в описании пробы URL /_healthz на /_health
```
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  105s               default-scheduler  Successfully assigned default/frontend-c8699877c-sbv6p to kind-worker2
  Normal   Pulled     105s               kubelet            Container image "raacharg/hipster-frontend:v0.0.2" already present on machine
  Normal   Created    105s               kubelet            Created container server
  Normal   Started    105s               kubelet            Started container server
  Warning  Unhealthy  5s (x11 over 95s)  kubelet            Readiness probe failed: Get "http://10.244.3.16:8080/_healthz": dial tcp 10.244.3.16:8080: connect: connection refused
```
- Првоерил отслеживание
```
kubectl rollout status deployment/frontend
Waiting for deployment "frontend" rollout to finish: 1 out of 3 new replicas have been updated...
```
- вернул манифест в рабочее состояние.

18. DaemonSet | Задание со звездочкой и DaemonSet | Задание с двумя звездочками.
- Создал и применил ds. Сразу с tolerations для развертывания на всех узлах кластера.
```
kubectl get pods -n monitoring 
NAME                  READY   STATUS    RESTARTS   AGE
node-exporter-2wlbk   2/2     Running   0          3m4s
node-exporter-k9hzs   2/2     Running   0          3m4s
node-exporter-m9sw5   2/2     Running   0          3m4s
node-exporter-nw8n5   2/2     Running   0          3m4s
node-exporter-p59xd   2/2     Running   0          3m4s
node-exporter-txzh5   2/2     Running   0          3m4s
```
- Проверил проброс порта, все работает, но вывод громоздкий, тут его не привожу.
