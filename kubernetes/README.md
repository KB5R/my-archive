# Гайд по управлению Kubernetes

## Command 
- Просмотр подов (-A означает AllNamespace)
```
kubectl get pods
```
- Просмотр подов и статусов со всеми labels
```
kubectl get pods --show-labels
```

- просмотр всех нод кластера
```
kubectl get nodes
```
- Запуск пода через command line
```
kubectl run app-kuber-1 --image=nginx:latest --port=80
```
- Запуск пода(deploy,manifest) через файл
```
kubectl apply -f kuber-pod.yaml
```
- Пример просто manifest file c labels
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-kuber-2
  labels:
    enviroment: dev
    app: http-server
spec:
  containers:
  - name: app-kuber-3-label
    image: nginx:latest
    ports:
    - containerPort: 80
```
- Проверка подробной информации о (pods, nodes)
```
kubectl describe pods app-kuber-1
kubectl describe node work2
```
- Как запустить выпустить под во внешку с помощью port-forward
```
kubectl port-forward --address 0.0.0.0  app-kuber-1 80:80
```
- Как добавить labels к node?
```
kubectl label node work2 gpu=true
kubectl get nodes --show-labels
```
- Проверить все namespaces
```
kubectl get ns
```
- Как добавить новый namespaces
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```
```
kubectl create namespace qa
```
- Как запустить под с опеределенным namespaces
```
apiVersion: v1
kind: Pod
metadata:
  name: app-kuber-dev-1
  namespace: dev # <-- namespaces
  labels:
    app: http-server
spec:
  containers:
  - name: httpd
    image: httpd:latest
    ports:
    - containerPort: 80
```
```
kubectl get pods -n dev
NAME              READY   STATUS    RESTARTS   AGE
app-kuber-dev-1   1/1     Running   0          16m
```

### Работа с Replicaset and ReplicationController

- ReplicationController
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: kuber-rc
spec:
  replicas: 3
  selector:
    app: kuber
  template:
    metadata:
      name: kuber-app
      labels:
        app: kuber
    spec:
      containers:
      - name: kuber-app-image
        image: nginx:latest
        ports:
        - containerPort: 80
```
- Проверить RC
```
kubectl get rc
```
- Смотрим pods
```
kubectl get pods -l app=kuber
NAME             READY   STATUS    RESTARTS   AGE
kuber-rc-5lhr8   1/1     Running   0          10m
kuber-rc-dq5qj   1/1     Running   0          10m
kuber-rc-srg27   1/1     Running   0          10m
```

### Работа с Deployment
- Создание deployment
```
kubectl create deployment kuber-ctl-app --image=nginx:latest --port=80 --replicas=3
```
```
kubectl get deploy
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
kuber-ctl-app   3/3     3            3           43s
```
- Получить всю информацию о поде
```
kubectl get po kuber-ctl-app-5c5559b976-b9v7p -o yaml
```
- Deplyment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuber
  labels:
    app: kuber
spec:
  replicas: 5
  minReadySeconds: 10
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
    type: RollingUpdate
  selector:
    matchLabels:
      app: http-server
  template:
    metadata:
      labels:
        app: http-server
    spec:
      containers:
      - name: kuber-app
        image: bokovets/kuber:v2.0
        ports:
        - containerPort: 8000
```

### 🔢 `replicas: 5`

Определяет количество Pod'ов, которые Kubernetes должен держать в работающем состоянии.

В этом случае: **5 Pod'ов одновременно**.

---

### ⏱ `minReadySeconds: 10`

После того как Pod перейдёт в состояние `"Ready"`, Kubernetes подождёт **10 секунд**, прежде чем считать его реально готовым к работе.

Полезно при обновлениях — позволяет убедиться, что под стабилен, прежде чем продолжать `rolling update`.

---

### 🔄 `strategy`

Настройка стратегии обновления Pod'ов.

#### `type: RollingUpdate`

Обновление происходит **поэтапно**, без полной остановки приложения.

Kubernetes создаёт **новые Pod'ы и удаляет старые постепенно**.

#### `rollingUpdate` параметры:

```
maxSurge: 1
Во время обновления можно создать на 1 Pod больше, чем указано в replicas.
Например: если указано replicas: 5, то во время обновления может быть до 6 Pod'ов.

maxUnavailable: 1
Разрешает, чтобы на 1 Pod меньше было доступно во время обновления.
То есть, если один старый Pod выключен, а новый ещё не готов — это допустимо.
```
- Полезная команда!!!
```bash
while true; do curl 192.168.49.2:32461; sleep 2; echo; done
```

