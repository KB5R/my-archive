# Настройка пользователей в Linux

## Создание пользователей

Чтобы создать нового пользователя в Linux с домашней директорией, используйте команду:

```bash
sudo useradd -m -s /bin/bash username
```

- `-m` — создаёт домашнюю директорию (`/home/username`)
- `-s /bin/bash` — указывает оболочку Bash

После создания пользователя задайте пароль:

```bash
sudo passwd username
```

## Добавление в группу sudo

Чтобы дать пользователю возможность использовать `sudo`, добавьте его в группу `sudo` (для Debian/Ubuntu) или `wheel` (для CentOS/RHEL/Fedora):

```bash
# Для Debian/Ubuntu
sudo usermod -aG sudo username

# Для CentOS/RHEL/Fedora
sudo usermod -aG wheel username
```

После этого пользователь сможет выполнять команды с привилегиями `sudo`.

## Доступ к root

### Включение пользователя root

В некоторых дистрибутивах root-пользователь по умолчанию отключён. Чтобы задать ему пароль и активировать его:

```bash
sudo passwd root
```

После этого можно переключиться на root с помощью:

```bash
su -
```

### Разрешение пользователю выполнять `su`

Если при выполнении `su` появляется ошибка «Authentication failure», убедитесь, что пользователь входит в группу `wheel` (для RHEL/Fedora) или `sudo` (для Debian/Ubuntu):

```bash
sudo usermod -aG wheel username  # RHEL/Fedora
sudo usermod -aG sudo username   # Debian/Ubuntu
```

Дополнительно, в некоторых системах необходимо отредактировать `/etc/pam.d/su` и раскомментировать строку:

```bash
# auth required pam_wheel.so use_uid
```

## Проверка прав

Чтобы проверить, в каких группах состоит пользователь, используйте команду:

```bash
groups username
```

Чтобы проверить, есть ли у пользователя доступ к `sudo`, выполните:

```bash
sudo -l -U username
```

## Удаление пользователя

Чтобы удалить пользователя и его домашнюю директорию:

```bash
sudo userdel -r username
```
