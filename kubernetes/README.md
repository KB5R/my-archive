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

### Работа с Deployment LESSON9
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

## Отличие Rolling Update от Recreate в Kubernetes

### 1. Rolling Update (Постепенное обновление)
**Как работает:**  
- Новые поды создаются постепенно, а старые удаляются только после успешного запуска новых.  
- Параллельно могут работать как старые, так и новые версии подов.  
- Kubernetes контролирует количество доступных подов, чтобы не было простоев.  

**Плюсы:**  
- Без простоя (zero-downtime) — приложение остается доступным во время обновления.  
- Плавное обновление — если что-то пойдет не так, можно откатиться.  

**Минусы:**  
- На время обновления могут одновременно работать две версии приложения, что может вызвать проблемы с совместимостью (например, если API изменилось).  
- Обновление занимает больше времени, чем `Recreate`.  

**Настройка в Deployment:**  
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # Сколько дополнительных подов можно создать
    maxUnavailable: 0  # Сколько подов может быть недоступно во время обновления#
```


## 2. Recreate (Полное пересоздание)

### Принцип работы
- **Полное удаление** всех старых подов перед созданием новых
- **Короткий простой (downtime)** - приложение недоступно между удалением старых и готовностью новых подов
- **Атомарное обновление** - в любой момент времени работает только одна версия приложения

### Преимущества
✅ **Гарантированная согласованность** - исключены конфликты между разными версиями  
✅ **Простота реализации** - не требует сложной логики балансировки  
✅ **Быстрое завершение** - не тратится время на постепенное обновление  
✅ **Предсказуемость** - полный контроль над процессом обновления  

### Ограничения
⚠️ **Время простоя** - сервис недоступен во время обновления  
⚠️ **Нет плавного перехода** - возможны ошибки при высокой нагрузке после включения  
⚠️ **Нет отката "на лету"** - для возврата нужно повторное развертывание  

### Конфигурация Deployment
```yaml
spec:
  strategy:
    type: Recreate

```

### Проверка историй обновлений

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuber
  labels:
    app: kuber
  annotations: # Это необходимо для kubectl rollout history deployment/<deployment-name>
    kubernetes.io/change-cause: "Deploy v2.0 with new response handlers"  # Для истории изменений
spec:
```

- `kubectl rollout history deployment/kuber` - это история
```yaml
REVISION  CHANGE-CAUSE
8         Deploy v4.0(test) with new response handlers
9         Deploy v10.0(test) with new response handlers
10        Deploy v10.0(nginx) with new response handlers
```

- Если надо срочно откатится назад после обновления
```
kubectl rollout undo deployment/kuber
```
- Откатится на опреденный апдейт 
```
kubectl rollout undo deployment/kuber --to-revision=8
```

# LESSON10

# Есть 3 основных service
- ClusterIP
- NodePort
- LoadBalancer
**Так же есть допы**
- ExternalName

**Как по мне самая удобная и понятная в реализации это nodeport**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuber
  labels:
    app: kuber
spec:
  replicas: 3
  minReadySeconds: 5
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
        image: bokovets/kuber:v1.0
        ports:
        - containerPort: 8000
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kuber-service-nodeport
spec:
 # externalTrafficPolicy: Local
  # sessionAffinity: ClientIP -- Полезный папрмаер работает так когда бразуер потом k8s кэширует запрос service всегда будет отпровлятье его на один и тот же под
  
  selector:
    app: http-server
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
      nodePort: 30080 # port-range: 30000-32767
  type: NodePort

```
**Service в данном случае понимаю куда надо идти с помощью**
```
  selector:
    app: http-server
```
**После создания service видим след*
```
kubectl get svc
NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kuber-service-nodeport   NodePort    10.233.44.61   <none>        80:30080/TCP   11m
```
**В моей конфигурации ноды не имеют белых ip но если проверить ip и порт**
```
❯ curl 10.236.0.191:30080
V1.0 - Hello world from hostname: kuber-cc68f9f49-898c9%
```
