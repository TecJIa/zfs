# homework-ZFS

Описание домашнего задания
---
1. Определить алгоритм с наилучшим сжатием
2. Определить настройки пула
3. Работа со снапшотами

---
- Этап 0: Подготовка для выполнения задания

Разворачиваем виртуалку посредствам Vagrant. Получаем ВМ с дополнительными дисками и предустановленным zfs

ОС для настройки: Ubuntu 22.04.4 LTS (в методичке предлагался centos7, решил попробовать другую, в целом, отличается только установкой самого zfs)

Vagrant версии 2.4.1

VirtualBox версии 7.0.18

*Ремарка, в методичке выделяется 512Мб памяти для создаваемых дисков, я выделил меньше в силу ограниченных ресурсов основной совей машины. 

---
- Этап 1: Определить алгоритм с наилучшим сжатием

Смотрим список всех дисков

```bash
lsblk
```  
![images2](./images/zfs_1.png)

Создаём пул из двух дисков в режиме RAID 1

```bash
zpool create otus1 mirror /dev/sdc /dev/sdd
zpool create otus2 mirror /dev/sde /dev/sdf
zpool create otus3 mirror /dev/sdg /dev/sdh
zpool create otus4 mirror /dev/sdi /dev/sdj
```  
![images2](./images/zfs_2.png)




Смотрим информацию о пулах

```bash
zpool list
```  
![images2](./images/zfs_3.png)


И их статусы

```bash
zpool status
```  
![images2](./images/zfs_4.png)


Добавляем разные алгоритмы сжатия в разные файловые системы

```bash
zfs set compression=lzjb otus1
zfs set compression=lz4 otus2
zfs set compression=gzip-9 otus3
zfs set compression=zle otus4
```  
![images2](./images/zfs_5.png)


Проверяем (обращаем внимание на методы сжатия)

```bash
zfs get all | grep compression
```  
![images2](./images/zfs_6.png)


Качаем одинаковый файл во все пулы, проверяем

```bash
# команда из методички без изменений, на убунте тоже работает (круто, использование цикла, запомнить)
for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done
```  
![images2](./images/zfs_7.png)

```bash
ls -l /otus*
```  
![images2](./images/zfs_8.png)

Проверяем занятое место в разных пулах (помним, что у них разные методы сжатия) 

```bash
zfs list
```  
![images2](./images/zfs_9.png)

Смотрим компрессию 

```bash
zfs get all | grep compressratio | grep -v ref
```  
![images2](./images/zfs_10.png)


```bash
# Делаем вывод, что самый наилучший метод сжатия имеет алгоритм gzip-9
```  

---

- Этап 2: Определить настройки пула.

Скачиваем архив и распаковываем его

```bash
wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'
```  
![images2](./images/zfs_11.png)

```bash
tar -xzvf archive.tar.gz
```  
![images2](./images/zfs_12.png)


Проверяем возможность импортирования скаченного архива. 

```bash
zpool import -d zpoolexport/
# Имя: otus
# Тип рэйда: mirror-0
# Состав: filea + fileb
```  
![images2](./images/zfs_13.png)


Импортируем пул себе в ОС, проверяем статус. 

```bash
zpool import -d zpoolexport/ otus
zpool status
```  
![images2](./images/zfs_14.png)


Проверяем все доступные параметры файловой системы. 

```bash
zfs get all otus
```  
![images2](./images/zfs_15.png)


Проверяем нужные нам параметры файловой системы. 

```bash
zfs get available otus
```  
![images2](./images/zfs_16.png)

```bash
zfs get readonly otus
```  
![images2](./images/zfs_17.png)

```bash
zfs get recordsize otus
```  
![images2](./images/zfs_18.png)

```bash
zfs get compression otus
```  
![images2](./images/zfs_19.png)

```bash
zfs get checksum otus
```  
![images2](./images/zfs_20.png)

---

- Этап 3: Работа со снапшотом, поиск сообщения от преподавателя.

Скачиваем файл задания 

```bash
wwget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download
# Тут чего-то консоль немного подзалипла, пришлось пнуть
```  
![images2](./images/zfs_21.png)


Восстановление ФС из снапшота 

```bash
zfs receive otus/test@today < otus_task2.file
# Стоит отметить, что весьма простой процесс восстановления 
```  
![images2](./images/zfs_22.png)


Ищем в каталоге /otus/test файл с именем “secret_message” 

```bash
find /otus/test -name "secret_message"
```  
![images2](./images/zfs_23.png)


Нашли 

```bash
nano /otus/test/task1/file_mess/secret_message
```  
![images2](./images/zfs_24.png)














