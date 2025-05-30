
# Полное руководство по работе с LVM в Linux

Logical Volume Manager (LVM) — это система управления разделами и дисками, которая позволяет более гибко работать с хранилищем данных. Она предоставляет возможности динамического изменения размеров томов, создания снимков и управления пространством.

---

## Часть 1: Основные концепции LVM

### Основные термины

1. **Physical Volume (PV)** — физический диск или раздел, используемый для создания LVM.
2. **Volume Group (VG)** — группа, объединяющая несколько физических томов (PVs).
3. **Logical Volume (LV)** — логический том, на котором создаются файловые системы.
4. **Physical Extents (PE)** — минимальный блок данных на уровне физического тома, который используется для распределения данных в VG.

### Преимущества LVM

- Гибкое управление размером томов.
- Снимки (snapshots) для резервного копирования.
- Простота расширения файловой системы.
- Возможность добавления новых дисков в существующую группу.

---

## Часть 2: Установка и настройка LVM

### Шаг 1: Установка пакетов

На большинстве дистрибутивов LVM устанавливается по умолчанию. Если требуется, можно установить:

```bash
apt install lvm2  # Для Ubuntu/Debian
dnf install lvm2  # Для Fedora
```

### Шаг 2: Проверка доступных дисков

Перед началом работы убедитесь, какие устройства доступны:

```bash
lsblk
fdisk -l
```

### Шаг 3: Создание Physical Volume (PV)

Инициализируйте физический диск или раздел:

```bash
pvcreate /dev/sdX  # Например, /dev/sdb
```

Проверьте созданный PV:

```bash
pvs
```

### Шаг 4: Создание Volume Group (VG)

Объедините один или несколько PV в Volume Group:

```bash
vgcreate my_vg /dev/sdX
```

Проверьте созданные VG:

```bash
vgs
```

### Шаг 5: Создание Logical Volume (LV)

Создайте логический том:

```bash
lvcreate -L 10G -n my_lv my_vg
```

Или используйте весь доступный объем:

```bash
lvcreate -l 100%FREE -n my_lv my_vg
```

Проверьте созданные LV:

```bash
lvs
```

### Шаг 6: Создание файловой системы и монтирование

Создайте файловую систему на логическом томе:

```bash
mkfs.ext4 /dev/my_vg/my_lv
```

Создайте точку монтирования и примонтируйте том:

```bash
mkdir /mnt/my_data
mount /dev/my_vg/my_lv /mnt/my_data
```

Проверьте монтирование:

```bash
df -h
```

---

## Часть 3: Усложняем работу с LVM

### Расширение Logical Volume

1. Добавьте новый диск и создайте PV:
   ```bash
   pvcreate /dev/sdY
   ```
2. Добавьте PV в существующую VG:
   ```bash
   vgextend my_vg /dev/sdY
   ```
3. Увеличьте размер LV:
   ```bash
   lvextend -L+5G /dev/my_vg/my_lv
   ```
   Или используйте весь доступный объем:
   ```bash
   lvextend -l +100%FREE /dev/my_vg/my_lv
   ```
4. Расширьте файловую систему:
   - Для ext4:
     ```bash
     resize2fs /dev/my_vg/my_lv
     ```
   - Для XFS:
     ```bash
     xfs_growfs /mnt/my_data
     ```

### Снимки (Snapshots)

1. Создайте снимок:
   ```bash
   lvcreate -L 1G -s -n my_snapshot /dev/my_vg/my_lv
   ```
2. Восстановление из снимка:
   ```bash
   lvconvert --merge /dev/my_vg/my_snapshot
   ```
3. Удаление снимка:
   ```bash
   lvremove /dev/my_vg/my_snapshot
   ```

### Уменьшение Logical Volume

1. Отмонтируйте логический том:
   ```bash
   umount /mnt/my_data
   ```
2. Сначала уменьшите файловую систему:
   ```bash
   resize2fs /dev/my_vg/my_lv 5G
   ```
3. Уменьшите LV:
   ```bash
   lvreduce -L 5G /dev/my_vg/my_lv
   ```
4. Примонтируйте обратно:
   ```bash
   mount /dev/my_vg/my_lv /mnt/my_data
   ```

---

## Часть 4: Резервное копирование и восстановление

### Резервное копирование VG

Экспортируйте конфигурацию:

```bash
vgcfgbackup
```

### Восстановление VG

Если VG была повреждена, выполните восстановление:

```bash
vgcfgrestore my_vg
```

---

## Часть 5: Диагностика и мониторинг

### Проверка состояния LVM

1. Список всех томов:
   ```bash
   lvs
   vgs
   pvs
   ```
2. Проверка состояния томов:
   ```bash
   lvdisplay
   vgdisplay
   pvdisplay
   ```

### Удаление LV, VG и PV

1. Удалите логический том:
   ```bash
   lvremove /dev/my_vg/my_lv
   ```
2. Удалите группу томов:
   ```bash
   vgremove my_vg
   ```
3. Удалите физический том:
   ```bash
   pvremove /dev/sdX
   ```

---

## Заключение

LVM предоставляет мощные инструменты для управления хранилищем данных. С помощью этого руководства вы сможете создавать, изменять и управлять дисковым пространством на сервере, выполняя как базовые, так и сложные задачи.
