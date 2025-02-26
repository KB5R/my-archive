# Настройка и добавление записей A и PTR в BIND9

## 1️⃣ Добавление A-записи (IP → Имя)
### Пример: Добавим `web.solutions.local` с IP `192.168.0.200`.

### 🔹 1. Открываем зону `solutions.local`
Редактируем файл прямой зоны:
```bash
sudo nano /etc/bind/db.solutions.local
```
Добавляем в конец файла:
```dns
web     IN  A   192.168.0.200
```
**Объяснение:**  
- `web` — имя хоста (`web.solutions.local`).  
- `IN A` — A-запись (IP-адрес).  
- `192.168.0.200` — IP-адрес для `web.solutions.local`.  

Сохраняем (`CTRL+X`, затем `Y` и `ENTER`).

---

## 2️⃣ Добавление PTR-записи (Имя → IP)
### Пример: Привяжем `192.168.0.200` к `web.solutions.local`.

### 🔹 1. Открываем обратную зону (`PTR`)
Редактируем файл обратной зоны:
```bash
sudo nano /etc/bind/db.192.168.0
```
Добавляем в конец файла:
```dns
200     IN  PTR  web.solutions.local.
```
**Объяснение:**  
- `200` — последний октет IP-адреса (`192.168.0.200`).  
- `IN PTR` — PTR-запись (обратное разрешение).  
- `web.solutions.local.` — имя, на которое будет резолвиться IP.  

⚠️ **Важно:** Не забудь точку в конце `web.solutions.local.`!

Сохраняем (`CTRL+X`, затем `Y` и `ENTER`).

---

## 3️⃣ Обновляем Serial в файлах зон
Каждый раз при изменении файла зоны **нужно увеличивать** номер `Serial`.

Находим в обоих файлах (`db.solutions.local` и `db.192.168.0`) строку вида:
```dns
2025022601  ; Serial
```
**Увеличиваем число** (например, `2025022602`).

---

## 4️⃣ Проверяем конфигурацию
Перед перезапуском проверяем ошибки:
```bash
sudo named-checkconf
sudo named-checkzone solutions.local /etc/bind/db.solutions.local
sudo named-checkzone 0.168.192.in-addr.arpa /etc/bind/db.192.168.0
```
Если ошибок нет, **перезапускаем BIND**:
```bash
sudo systemctl restart bind9  # (Debian/Ubuntu)
sudo systemctl restart named  # (RHEL/Fedora)
```

---

## 5️⃣ Проверяем работу  
### 🔹 1. Проверяем A-запись
```bash
nslookup web.solutions.local
```
Или:
```bash
dig web.solutions.local
```
Ожидаемый результат:
```dns
web.solutions.local.  86400   IN  A   192.168.0.200
```

### 🔹 2. Проверяем PTR-запись
```bash
nslookup 192.168.0.200
```
Или:
```bash
dig -x 192.168.0.200
```
Ожидаемый результат:
```dns
200.0.168.192.in-addr.arpa. 86400 IN  PTR web.solutions.local.
```

---

✅ **Теперь `web.solutions.local` работает и в прямом (`A`), и в обратном (`PTR`) разрешении!** 🚀  
Если нужно добавить ещё хосты, просто повторяйте те же шаги!

