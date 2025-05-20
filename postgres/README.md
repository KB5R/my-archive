# Информация работы с Postgres DBA1
* Ubuntu 22.04.05
## 1.	Установка и управление сервером
- Установка пакетным менеджером
```
# Import the repository signing key:
sudo apt install curl ca-certificates
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc

# Create the repository configuration file:
. /etc/os-release
sudo sh -c "echo 'deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] https://apt.postgresql.org/pub/repos/apt $VERSION_CODENAME-pgdg main' > /etc/apt/sources.list.d/pgdg.list"

# Update the package lists:
sudo apt update

# Install the latest version of PostgreSQL:
# If you want a specific version, use 'postgresql-17' or similar instead of 'postgresql'
sudo apt -y install postgresql-16
```
- C какими параметрами был собран сервер
```
/usr/lib/postgresql/16/bin/pg_config --configure
```

- Остановка
```
sudo pg_ctlcluster 16 main stop
```
- Информация
```
pg_lsclusters 16 main
```
- Перезапуск
```
sudo pg_ctlcluster 16 main restart
```
- Перечитать конфиг
```
sudo pg_ctlcluster 16 main reload
```
- Статус
```
sudo pg_ctlcluster 16 main status
```
- Лог
```
tail -n 10 /var/log/postgresql/postgresql-16-main.log
```