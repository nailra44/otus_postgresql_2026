Домашнее задание
Перенос данных PostgreSQL

Цель:
перенести каталог данных PostgreSQL на дополнительный диск и восстановить работоспособность кластера при нехватке места на системном диске;


Описание/Пошаговая инструкция выполнения домашнего задания:

Контекст
Команда готовит стенд для нагрузочного тестирования нового релиза: ожидаются длительные прогоны и большой объём данных, чтобы проверить поведение системы при росте таблиц и интенсивной записи.
 На системном диске ВМ места недостаточно и его нельзя быстро расширить без рисков для стенда. Поэтому под данные PostgreSQL выделяется отдельный диск, который можно масштабировать независимо и при необходимости переносить между инстансами. Задача - перенести каталог данных PostgreSQL на новый том, обеспечить автомонтирование после перезагрузки и вернуть кластер в рабочее состояние. Успех подтверждается тем, что кластер запускается с новым путём к данным, а тестовая таблица и строки остаются на месте.
Описание / пошаговая инструкция выполнения ДЗ
1) Создайте виртуальную машину с Ubuntu 22.04/24.04 LTS в ЯО/VirtualBox (или эквивалентный стенд с возможностью подключать диск);
   - выполнено
2) Установите PostgreSQL через apt;
   - выполнено
3) Проверьте, что кластер запущен (pg_lsclusters);
   - выполнено
4) Под пользователем postgres подключитесь в psql, создайте таблицу test и добавьте 1–2 строки;
   - выполнено
5) Остановите кластер PostgreSQL штатной командой (pg_ctlcluster ... stop);
   - выполнено
6) Создайте дополнительный диск 10 GB и подключите его к виртуальной машине;
   - выполнено
7) Разметьте диск, создайте файловую систему и примонтируйте, например, в /mnt/data;
   - выполнено
8) Настройте автомонтирование после перезагрузки (через fstab) и проверьте, что диск остаётся примонтированным;
   - выполнено
9) Назначьте владельцем каталога /mnt/data пользователя postgres;
   - выполнено
10) Перенесите каталог данных PostgreSQL с текущего пути на /mnt/data (с сохранением прав);
   - выполнено
11) Попробуйте запустить кластер; зафиксируйте результат и причину (если не стартовал);
   - выполнено
12) Найдите в конфигурации /etc/postgresql/<версия>/main параметр, который указывает на каталог данных, измените его на новый путь и объясните изменение;
13) Запустите кластер повторно;
   - выполнено
14) Подключитесь через psql и проверьте, что таблица test и данные сохранились;
   - выполнено
   
15) Задание со звёздочкой: перенесите внешний диск на новую ВМ и запустите PostgreSQL на новой ВМ, используя данные на этом диске; опишите шаги и итоговый результат;

Установка PostgreSQL через apt
```sh
sudo atp install postgresql-common
    # проверка установки
    gorskiy-nn@gorskiy-nn-VirtualBox:~$ dpkg -l postgresql-common
    ┌── Desired=Unknown/Install/Remove/Purge/Hold
    │┌─ Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
    ││┌ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
    │││ Name              Version      Architecture Description
    ├┼┼─═════════════════─════════════─════════════─═══════════════════════════════════
    ii  postgresql-common 290ubuntu1   all          PostgreSQL database-cluster manager
# подключение репозитория 
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
    # проверяем подключение репозитория 
    ls -l /etc/apt/sources.list.d/ | grep -i pgdg

# утилита просмотра кластеров
pg_lsclusters
    # результат выполнения
    gorskiy-nn@gorskiy-nn-VirtualBox:~$ pg_lsclusters
    Ver Cluster Port Status Owner    Data directory              Log file
    17  main    5432 online postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log
        # удаляем кластер main установленный по умолчанию
        sudo pg_dropcluster --stop 17 main
            # результат выполнения
            gorskiy-nn@gorskiy-nn-VirtualBox:~$ pg_lsclusters
            Ver Cluster Port Status Owner Data directory Log file
# установка конкретной версии postgresql
    # устанавливаем бинарники
    sudo apt install postgresql-18
       # ПРоверка установки пакета
       dpkg -l postgresql-18
           ┌── Des           ired=Unknown/Install/Remove/Purge/Hold
           │┌─ Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
           ││┌ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
           │││ Name           Version            Architecture Description
           ├┼┼─══════════════─══════════════════─════════════─═════════════════════════════════════════════════════════
           ii  postgresql-18  18.6-1.pgdg26.04+2 amd64        The World's Most Advanced Open Source Relational Database
```
Установка кластера
```sh
sudo pg_createcluster 18 pg_18 --port 5456
    # результат выполнения
    Creating new PostgreSQL cluster 18/pg_18 ...
        # Просмотр кластера
        pg_lsclusters
            Ver Cluster Port Status Owner    Data directory               Log file
            18  pg_18   5456 down   postgres /var/lib/postgresql/18/pg_18 /var/log/postgresql/postgresql-18-pg_18.log

sudo pg_ctlcluster start 18 pg_18
        # Просмотр кластера
        pg_lsclusters
            Ver Cluster Port Status Owner    Data directory               Log file
            18  pg_18   5456 online postgres /var/lib/postgresql/18/pg_18 /var/log/postgresql/postgresql-18-pg_18.log
```            
подключение postgres
```sh
sudo -u postgres psql --port 5456    
   #результат
   psql (18.6 (Ubuntu 18.6-1.pgdg26.04+2))
   Type "help" for help.
   postgres=#
       # задать пароль для postgres
       \password

```
Редактирование файлов конфигурации
```sh
    postgres=# SHOW hba_file;
        #результат
                       hba_file
        --------------------------------------
         /etc/postgresql/18/pg_18/pg_hba.conf
        (1 row)

        postgres=# SHOW config_file;
                       config_file
        ------------------------------------------
         /etc/postgresql/18/pg_18/postgresql.conf
        (1 row)
    sudo nano /etc/postgresql/18/pg_18/pg_hba.conf
        #результат
        sudo cat /etc/postgresql/18/pg_18/pg_hba.conf | grep -A1  'IPv4 local connections'
            # IPv4 local connections:
            host    all             all             all                     scram-sha-256
    sudo nano /etc/postgresql/18/pg_18/postgresql.conf
        #результат
        sudo cat /etc/postgresql/18/pg_18/postgresql.conf | grep liste
        listen_addresses = '*'          # what IP address(es) to listen on;

```
Создание базы, таблиц
```sh 
        #создание базы
       postgres=# 
       CREATE DATABASE newtest;
           #результат
           CREATE DATABASE
               \c newtest
               #создание таблицы
               CREATE TABLE first_name(id int, name text);
                   #результат
                   CREATE TABLE
                       #заполнение данными таблицы
                       INSERT INTO first_name (id, name) VALUES (1, 'Grisha');
                       INSERT INTO first_name (id, name) VALUES (2, 'Vasya');
                       INSERT INTO first_name (id, name) VALUES (3, 'Serega');
                       INSERT INTO first_name (id, name) VALUES (4, 'Bred');
                           #результат
                           INSERT 0 1
                           INSERT 0 1
                           INSERT 0 1
                           INSERT 0 1
                                newtest=# select * from first_name;
                                #результат        
                                    id |  name
                                    ----+--------
                                      1 | Grisha
                                      2 | Vasya
                                      3 | Serega
                                      4 | Bred
                                    (4 rows)
```
Остановка кластера PostgreSQL штатной командой
```sh
pg_lsclusters
    Ver Cluster Port Status Owner    Data directory               Log file
    18  pg_18   5456 online postgres /var/lib/postgresql/18/pg_18 /var/log/postgresql/postgresql-18-pg_18.log
sudo pg_ctlcluster 18 pg_18 stop
pg_lsclusters
    Ver Cluster Port Status Owner    Data directory               Log file
    18  pg_18   5456 down   postgres /var/lib/postgresql/18/pg_18 /var/log/postgresql/postgresql-18-pg_18.log
sudo pg_ctlcluster 18 pg_18 start
 pg_lsclusters
    Ver Cluster Port Status Owner    Data directory               Log file
    18  pg_18   5456 online postgres /var/lib/postgresql/18/pg_18 /var/log/postgresql/postgresql-18-pg_18.log



```
Создание виртуального диска
![](otusvd1.jpg)

Разметка диска, создание файловой системы
```sh
gorskiy-nn@gorskiy-nn-VirtualBox:~$ lsblk | grep sdb
sdb      8:16   0  11.2G  0 disk
# разметка
sudo fdisk /dev/sdb
    n	новый раздел
    p	primary (основной)
    1	номер раздела
    Enter	начало по умолчанию
    Enter	конец по умолчанию (весь диск)
    w	записать и выйти
 
        #результат
            gorskiy-nn@gorskiy-nn-VirtualBox:~$ lsblk | grep sdb
            sdb      8:16   0  11.2G  0 disk
            └─sdb1   8:17   0  11.2G  0 part

# создание файловой системы
sudo mkfs.ext4 /dev/sdb1
        #результат            
            lsblk -f | grep sdb
            sdb
            └─sdb1 ext4     1.0         dcb3f28b-6ddc-425c-b133-a473b7c0c8ee
# монтирование
sudo mkdir -p /mnt/data
sudo mount /dev/sdb1 /mnt/data
        #результат            
            df -h /mnt/data
            Filesystem      Size  Used Avail Use% Mounted on
            /dev/sdb1        11G  2.1M   11G   1% /mnt/dat
# Автомонтирование при загрузке
 sudo blkid /dev/sdb1
/dev/sdb1: UUID="dcb3f28b-6ddc-425c-b133-a473b7c0c8ee" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="518b671c-01"
    #добавить строку в конец файла /etc/fstab
sudo nano /etc/fstab
# строка
UUID=dcb3f28b-6ddc-425c-b133-a473b7c0c8ee  /mnt/data  ext4  defaults  0  2

sudo mount -a монтируем всё, что перечислено в /etc/fstab, но ещё не примонтировано.

проверка
df -h /mnt/data
```
смена владельца каталога

```sh
sudo chown -R postgres:postgres /mnt/data
```

Перенос каталога данных PostgreSQL с текущего пути на /mnt/data (с сохранением прав);

```sh
# просмотр каталога с данными
sudo -u postgres psql --port 5456 -h 10.62.11.89
    Password for user postgres:
    psql (18.6 (Ubuntu 18.6-1.pgdg26.04+2))
    SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
    Type "help" for help.
        postgres=# SHOW data_directory;
                data_directory
                ------------------------------
                /var/lib/postgresql/18/pg_18
                (1 row)
# перенос каталогов
sudo pg_ctlcluster 18 pg_18 stop
    pg_lsclusters
        Ver Cluster Port Status Owner    Data directory               Log file
        18  pg_18   5456 down   postgres /var/lib/postgresql/18/pg_18 /var/log/postgresql/postgresql-18-pg_18.log
            # копирование каталогов с сохранением прав
            sudo rsync -av /var/lib/postgresql/18/pg_18/ /mnt/data/
                # -a копирует все с сохранением дерева, атрибутов, прав
                # -v вывод для просмотра действий  
                    # посмотрим
                    ls -la /mnt/data/
                        # Результат
                        gorskiy-nn@gorskiy-nn-VirtualBox:/$ sudo ls -la /mnt/data/
                            total 104
                            drwx------ 20 postgres postgres  4096 Oct 11 16:07 .
                            drwxr-xr-x  3 root     root      4096 Oct 11 01:24 ..
                            -rw-------  1 postgres postgres     3 Oct 10 23:49 PG_VERSION
                            drwx------  7 postgres postgres  4096 Oct 11 15:38 base
                            drwx------  2 postgres postgres  4096 Oct 11 15:34 global
                            drwx------  2 postgres postgres 16384 Oct 11 01:21 lost+found
                            drwx------  2 postgres postgres  4096 Oct 10 23:49 pg_commit_ts
                            drwx------  2 postgres postgres  4096 Oct 10 23:49 pg_dynshmem
                            drwx------  4 postgres postgres  4096 Oct 11 16:07 pg_logical
                            drwx------  4 postgres postgres  4096 Oct 10 23:49 pg_multixact
                            drwx------  2 postgres postgres  4096 Oct 10 23:49 pg_notify
                            drwx------  2 postgres postgres  4096 Oct 10 23:49 pg_replslot
                            drwx------  2 postgres postgres  4096 Oct 10 23:49 pg_serial
                            drwx------  2 postgres postgres  4096 Oct 10 23:49 pg_snapshots
                            drwx------  2 postgres postgres  4096 Oct 11 16:07 pg_stat
                            drwx------  2 postgres postgres  4096 Oct 10 23:49 pg_stat_tmp
                            drwx------  2 postgres postgres  4096 Oct 10 23:49 pg_subtrans
                            drwx------  2 postgres postgres  4096 Oct 11 15:33 pg_tblspc
                            drwx------  2 postgres postgres  4096 Oct 10 23:49 pg_twophase
                            drwx------  4 postgres postgres  4096 Oct 11 15:40 pg_wal
                            drwx------  2 postgres postgres  4096 Oct 10 23:49 pg_xact
                            -rw-------  1 postgres postgres    88 Oct 10 23:49 postgresql.auto.conf
                            -rw-------  1 postgres postgres   132 Oct 11 12:48 postmaster.opts
                                # переименовать каталог
                                sudo mv /var/lib/postgresql/18/pg_18 /var/lib/postgresql/18/pg_18.old
                                    # создание символической ссылки
                                    sudo ln -s /mnt/data /var/lib/postgresql/18/pg_18
                                        # результат
                                        gorskiy-nn@gorskiy-nn-VirtualBox:/var/lib/postgresql/18$ ls -la
                                        total 12
                                        drwxr-xr-x  3 postgres postgres 4096 Oct 11 16:23 .
                                        drwxr-xr-x  3 postgres postgres 4096 Oct 11 15:37 ..
                                        lrwxrwxrwx  1 root     root        9 Oct 11 16:23 pg_18 -> /mnt/data
                                        drwx------ 19 postgres postgres 4096 Oct 11 16:07 pg_18.old
                                            #запуск кластера
                                             pg_lsclusters
                                                Ver Cluster Port Status Owner    Data directory               Log file
                                                18  pg_18   5456 down   postgres /var/lib/postgresql/18/pg_18 /var/log/postgresql/postgresql-18-pg_18.log
                                                sudo pg_ctlcluster 18 pg_18 start
                                                pg_lsclusters
                                                    Ver Cluster Port Status Owner    Data directory               Log file
                                                    18  pg_18   5456 online postgres /var/lib/postgresql/18/pg_18 /var/log/postgresql/postgresql-18-pg_18.log
 ```                                                   
изменение data_directory на новый путь                                                                                        
```sh
cat /etc/postgresql/18/pg_18/postgresql.conf | grep data_di
data_directory = '/var/lib/postgresql/18/pg_18'         # use data in another directory

cat /etc/postgresql/18/pg_18/postgresql.conf | grep data_di
#data_directory = '/var/lib/postgresql/18/pg_18'                # use data in another directory
data_directory = '/mnt/data'

    #перезапуск
    pg_lsclusters
         Ver Cluster Port Status Owner    Data directory Log file
         18  pg_18   5456 online postgres /mnt/data      /var/log/postgresql/postgresql-18-pg_18.log
    sudo pg_ctlcluster 18 pg_18 stop
    pg_lsclusters
         Ver Cluster Port Status Owner    Data directory Log file
         18  pg_18   5456 down   postgres /mnt/data      /var/log/postgresql/postgresql-18-pg_18.log
         sudo pg_ctlcluster 18 pg_18 sttart
         Error: Usage: /usr/bin/pg_ctlcluster <version> <cluster> <action> [-- <pg_ctl options>]
    sudo pg_ctlcluster 18 pg_18 start
    pg_lsclusters
         Ver Cluster Port Status Owner    Data directory Log file
         18  pg_18   5456 online postgres /mnt/data      /var/log/postgresql/postgresql-18-pg_18.log 
             # просмотр директории data_directory
             postgres=# SHOW data_directory;
                 data_directory
                ----------------
                 /mnt/data
                (1 row)
                # смотрим соханились ли данные
                    postgres=# \c newtest
                        SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
                        You are now connected to database "newtest" as user "postgres".                        newtest=# select * from first_name;
                         id |  name
                        ----+--------
                          1 | Grisha
                          2 | Vasya
                          3 | Serega
                          4 | Bred
                        (4 rows)

                        
```
```sh
# после копирования диска подключитьего в другой VM не получается - ошибка дублируется UUID
# установить новый
PS C:\Users\Murzik> & "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" internalcommands sethduuid "D:\HOME\VirtualBox VMs\otus-ubuntu2\otus-ubuntu_21.vdi"
UUID changed to: a1db14de-5b28-496d-bb1f-d4ed83f64e09

примонтировался /dev/sdb1        11G   65M   11G   1% /run/media/gorskiy-nn/dcb3f28b-6ddc-425c-b133-a473b7c0c8ee
# размонтируем
sudo umount /dev/sdb1
# монтируем
sudo mkdir -p /mnt/data
sudo mount /dev/sdb1 /mnt/data
# установка pg
    sudo apt update
    sudo atp install postgresql-common
    sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
    sudo apt install postgresql-18
        vi /etc/postgresql/18/pg_18/pg_hba.conf
            sudo cat /etc/postgresql/18/pg_18/pg_hba.conf | grep -A1 'IPv4 '
            # IPv4 local connections:
            host    all             all             all            scram-sha-256
        sudo vi /etc/postgresql/18/pg_18/postgresql.conf
            sudo cat /etc/postgresql/18/pg_18/postgresql.conf | grep -A1 'data_di'
                data_directory = '/mnt/data'            # use data in another directory
                                        # (change requires restart)
        sudo chown -R postgres:postgres /mnt/data
        sudo pg_ctlcluster 18 pg_18 start
        pg_lsclusters
            Ver Cluster Port Status Owner    Data directory Log file
            18  pg_18   5457 online postgres /mnt/data      /var/log/postgresql/postgresql-18-pg_18.log
        sudo -u postgres psql --port 5457
        \password

        postgres=# SHOW data_directory;
             data_directory
            ----------------
             /mnt/data
            (1 row)

            
        # проверяем перенеслась ли таблица
        # новый сервак
           inet 10.62.11.87/24 brd 10.62.11.255 scope global dynamic noprefixroute enp0s3

        sudo -u postgres psql --port 5457
            sql (18.6 (Ubuntu 18.6-1.pgdg26.04+2))
            Type "help" for help.

            postgres=# \l
                                                                 List of databases
               Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | Locale | ICU Rules |   Access privileges
            -----------+----------+----------+-----------------+-------------+-------------+--------+-----------+-----------------------
             app       | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           |
             newtest   | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           |
             postgres  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           |
             template0 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =c/postgres          +
                       |          |          |                 |             |             |        |           | postgres=CTc/postgres
             template1 | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =c/postgres          +
                       |          |          |                 |             |             |        |           | postgres=CTc/postgres
            (5 rows)

                postgres=# \c newtest
                You are now connected to database "newtest" as user "postgres".
                newtest=# select * from first_name;
                 id |  name
                ----+--------
                  1 | Grisha
                  2 | Vasya
                  3 | Serega
                  4 | Bred
                (4 rows)

```                          


