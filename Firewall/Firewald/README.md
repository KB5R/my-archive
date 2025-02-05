
# Руководство по работе с firewalld в AlmaLinux

## Содержание
1. [Введение](#введение)
2. [Установка и управление службой](#установка-и-управление-службой)
3. [Основные концепции](#основные-концепции)
    - [Зоны](#зоны)
    - [Службы](#службы)
    - [Порты](#порты)
4. [Основные команды](#основные-команды)
    - [Работа с зонами](#работа-с-зонами)
    - [Управление портами и службами](#управление-портами-и-службами)
5. [Богатые правила (Rich Rules)](#богатые-правила-rich-rules)
6. [Примеры настройки](#примеры-настройки)
7. [Логи и устранение неполадок](#логи-и-устранение-неполадок)
8. [Отключение firewalld](#отключение-firewalld)
9. [Дополнительные ресурсы](#дополнительные-ресурсы)

## Введение
Firewalld — это динамический брандмауэр для Linux, который позволяет управлять сетевыми правилами без перезагрузки системы. Он использует зоны для определения уровня доверия к сетевым интерфейсам и источникам трафика.

## Установка и управление службой

### Установка firewalld
```bash
sudo dnf install firewalld
```

### Запуск и автозагрузка
```bash
# Запустить firewalld
sudo systemctl start firewalld

# Включить автозапуск
sudo systemctl enable firewalld

# Проверить статус
sudo systemctl status firewalld
```

## Основные концепции

### Зоны
Зоны определяют уровень доверия к сетям. По умолчанию доступны следующие зоны:

| Название зоны | Уровень доверия | Описание |
|----------------|-----------------|----------|
| drop           | Низкий          | Все входящие соединения сбрасываются. |
| block          | Низкий          | Входящие соединения блокируются с уведомлением. |
| public         | Средний         | Для общедоступных сетей (по умолчанию в AlmaLinux). |
| external       | Средний         | Для внешних сетей с NAT. |
| internal       | Высокий         | Для доверенных внутренних сетей. |
| dmz            | Средний         | Для изолированных сетей (DMZ). |
| work           | Высокий         | Для рабочих сетей. |
| home           | Высокий         | Для домашних сетей. |
| trusted        | Максимальный    | Все соединения разрешены. |

### Службы
Службы — это предопределенные наборы правил для популярных приложений (например, http, ssh, ftp).

Список доступных служб:
```bash
sudo firewall-cmd --get-services
```

### Порты
Порты можно открывать вручную, если служба не определена.

## Основные команды

### Работа с зонами
- Просмотр активных зон:
  ```bash
  sudo firewall-cmd --get-active-zones
  ```

- Просмотр настроек зоны (например, public):
  ```bash
  sudo firewall-cmd --zone=public --list-all
  ```

- Изменение зоны для интерфейса (например, eth0 на internal):
  ```bash
  sudo firewall-cmd --zone=internal --change-interface=eth0 --permanent
  sudo firewall-cmd --reload
  ```

### Управление портами и службами
- Добавить службу:
  ```bash
  sudo firewall-cmd --zone=public --add-service=ssh --permanent
  ```

- Добавить порт:
  ```bash
  sudo firewall-cmd --zone=public --add-port=8080/tcp --permanent
  ```

- Удалить службу/порт:
  ```bash
  sudo firewall-cmd --zone=public --remove-service=ssh --permanent
  sudo firewall-cmd --zone=public --remove-port=8080/tcp --permanent
  ```

- Применить изменения:
  ```bash
  sudo firewall-cmd --reload
  ```

## Богатые правила (Rich Rules)
- Разрешить доступ по IP:
  ```bash
  sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.100" accept' --permanent
  ```

- Заблокировать IP:
  ```bash
  sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.101" reject' --permanent
  ```

- Удалить правило:
  ```bash
  sudo firewall-cmd --zone=public --remove-rich-rule='rule family="ipv4" source address="192.168.1.100" accept' --permanent
  ```

## Примеры настройки

### Пример 1: Веб-сервер (HTTP/HTTPS)
```bash
# Разрешить HTTP и HTTPS
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --zone=public --add-service=https --permanent
sudo firewall-cmd --reload
```

### Пример 2: Ограничение SSH по IP
```bash
# Запретить SSH для всех, кроме 192.168.1.100
sudo firewall-cmd --zone=public --remove-service=ssh --permanent
sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.100" service name="ssh" accept' --permanent
sudo firewall-cmd --reload
```

### Пример 3: Создание пользовательской зоны
```bash
# Создать зону "myzone"
sudo firewall-cmd --new-zone=myzone --permanent
sudo firewall-cmd --reload

# Настроить правила для myzone
sudo firewall-cmd --zone=myzone --add-service=ftp --permanent
sudo firewall-cmd --zone=myzone --add-port=9000/tcp --permanent
sudo firewall-cmd --reload
```

## Логи и устранение неполадок

- Просмотр логов firewalld:
  ```bash
  sudo journalctl -u firewalld
  ```

- Проверка правил:
  ```bash
  sudo firewall-cmd --list-all
  ```

- Диагностика блокировок:
  Используйте `tcpdump` или `nmap` для проверки доступности портов.

## Отключение firewalld

- Временно остановить:
  ```bash
  sudo systemctl stop firewalld
  ```

- Полностью отключить:
  ```bash
  sudo systemctl disable firewalld
  ```

### Важное
```
firewall-cmd --zone=public --set-target=DROP --permanent
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="IP" service name="ssh" accept' --permanent
firewall-cmd --zone=public --remove-service=http --permanent
firewall-cmd --zone=public --remove-rich-rule='rule family="ipv4" source address="0.0.0.0/0" port port="80" protocol="tcp" reject' --permanent
```

  

**Внимание!** Отключайте firewalld только при наличии альтернативного брандмауэра.

## Дополнительные ресурсы
- [Официальная документация](https://firewalld.org)
- Справочник по `firewall-cmd`: `man firewall-cmd`
