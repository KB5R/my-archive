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