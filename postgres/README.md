# Информация работы с Postgres DBA1
* Ubuntu 22.04.05
## 1. Установка и управление сервером
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
## 2. Использование psql
- Переключится на postgres
```
su - postgres
```
- Проверим текущее подключение
```
\conninfo
```
- Создать БД
```
CREATE DATABASE name
```
- Подключится к бд из psql
```
\c name
```
- Создание талблицы
```sql
CREATE TABLE courses(
tc_no text PRIMARY KEY,
title text,
hours integer
);
```
*Как это выглядит в человеческом понимании*


| Название столбца | Тип данных | Описание                     |
|------------------|------------|------------------------------|
| c_no             | text       | Номер курса (PRIMARY KEY)    |
| title            | text       | Название курса               |
| hours            | integer    | Продолжительность (в часах)  |


- Посмотреть список таблиц в базе
```
\dt
```
```
\dt
          List of relations
 Schema |   Name   | Type  |  Owner   
--------+----------+-------+----------
 public | courses  | table | postgres
 public | exams    | table | postgres
 public | students | table | postgres
(3 rows)
```
- Как наполнить таблицу
```sql
INSERT INTO courses(c_no, title, hours)
VALUES ('CS301', 'Базы данных', 30),
('CS305', 'Сети ЭВМ', 60);
```

- Посмотреть что в нутири таблицы
```
SELECT * FROM courses;
 c_no  |    title    | hours 
-------+-------------+-------
 CS301 | Базы данных |    30
 CS305 | Сети ЭВМ    |    60
(2 rows)
```
### Простые запросы
```
SELECT title, hours FROM courses;
    title    | hours 
-------------+-------
 Базы данных |    30
 Сети ЭВМ    |    60
(2 rows)
```