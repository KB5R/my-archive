### Планировка
#### Поместить `myMonitor.service` и `myMonitor.timer` в дерикторию `/etc/systemd/system`
`myMonitor.service`
```bash
[Unit]
Description=script_test_timer
Wants=myMonitor.timer

[Service]
Type=oneshot
ExecStart=/home/user/script.sh

[Install]
WantedBy=multi-user.target
```

`myMonitor.timer`
```bash
[Unit]
Description=Logs some system statistics to the systemd journal
Requires=myMonitor.service

[Timer]
Unit=myMonitor.service
OnCalendar=*-*-* *:*:00

[Install]
WantedBy=timers.target
```

### Проверка сервиса 
```
journalctl -S today -f -u myMonitor.service
```
