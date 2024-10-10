### Планировка
#### Поместить `myMonitor.service` и `myMonitor.timer` в дерикторию `/etc/systemd/system`

### Проверка сервиса 
```
journalctl -S today -f -u myMonitor.service
```
