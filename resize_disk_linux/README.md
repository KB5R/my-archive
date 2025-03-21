# Увеличения раздела /dev/sdaX без выключения:

### Шаг 1. Увеличте размер диска /dev/sdX любым способом и проверьте утелитой
```
fdisk -l /dev/sda
```
### Будет что то типа
```
GPT PMBR size mismatch (67108863 != 150994943) will be corrected by write.
The backup GPT table is not on the end of the device.
Disk /dev/sda: 72 GiB, 77309411328 bytes, 150994944 sectors
Disk model: QEMU HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 7F8FE12A-DBC9-434E-9485-DB62B6BA4336

Device     Start      End  Sectors Size Type
/dev/sda1   2048     4095     2048   1M BIOS boot
/dev/sda2   4096 67106815 67102720  32G Linux filesystem
```
### Шаг 2. Запуск fdisk для работы с диском
```
fdisk /dev/sda
```
### Шаг 3. Удаление раздела /dev/sda2
*Важно: Эти действия не удаляют данные, если вы не трогаете файловую систему, а только обновляете информацию о разделе.*
#### 1. Внутри fdisk напечатайте:
```
p
```
*Это выведет текущую таблицу разделов.*
#### 2. Удалите существующий раздел /dev/sda2 (старый раздел на 32 ГБ):
```
d
```
*Когда fdisk спросит номер раздела, выберите 2.*
#### 3. Проверьте таблицу снова, напечатав:
```
p
```
*Убедитесь, что остался только /dev/sda1.*
### Шаг 4. Создание нового увеличенного раздела
#### 1. Создайте новый раздел:
```
n
```
#### 2. Выберите primary (основной) раздел и укажите номер 2
#### 3. Укажите начальный сектор. Он должен совпадать с началом старого раздела, чтобы сохранить данные (в вашем случае это 4096):
#### 4. Когда fdisk попросит указать последний сектор, нажмите Enter, чтобы использовать все доступное пространство.
#### WARNING
```
Created a new partition 2 of type 'Linux filesystem' and of size 64 GiB.
Partition #2 contains a ext4 signature.

Do you want to remove the signature? [Y]es/[N]o: N
```
#### 5. Сохраните изменения и выйдите:
```
w
```
#### 6. Проверьте новую таблицу разделов:
```
p
```
### Шаг. 2 Расширение файловой системы.
#### Для ext4:
```
resize2fs /dev/sda2
```
#### Для xfs:
```
xfs_growfs /dev/sda2
```
### Шаг 6. Проверьте результат
```
df -h
lsblk
```
