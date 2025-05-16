# Гайд по управлению K8S

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
- Как добавить labels к node?
```
kubectl label node work2 gpu=true
kubectl get nodes --show-labels
```