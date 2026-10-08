Установка_postgresql

##TODO: Создайте ВМ с Ubuntu 22.04/24.04 или подготовьте хост, на котором будет развёрнут Docker;

- выполнено

TODO: Установите Docker Engine;

- выполнено

TODO: Создайте каталог для данных PostgreSQL на хосте: /var/lib/postgresql;
TODO: Разверните контейнер с PostgreSQL, смонтировав каталог хоста в каталог данных контейнера и пробросив порт 5432 для внешнего подключения;
TODO: Разверните контейнер с клиентом PostgreSQL (psql);
TODO: Подключитесь из контейнера с клиентом к контейнеру с сервером; создайте таблицу orders_test и добавьте минимум 2 строки;
TODO: Подключитесь к PostgreSQL с ноутбука/рабочего компьютера извне хоста (по адресу хоста и порту 5432); выполните проверочный select из таблицы orders_test;
TODO: Остановите и удалите контейнер с сервером PostgreSQL;
TODO: Создайте контейнер с сервером заново, используя тот же смонтированный каталог данных;
TODO: Подключитесь повторно из контейнера с клиентом и извне; проверьте, что строки в orders_test сохранились;

Установка PostgreSQL
```sh
# Создание структуры каталогов через которую можно управлять кластерами postgresql, устанавливает утилиты управления кластером.
sudo apt install -y postgresql-common 

# подключение репозитория 
sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
    # проверяем подключение репозитория 
    ls -l /etc/apt/sources.list.d/ | grep -i pgdg
    # результат
    -rw-r--r-- 1 root root  186 Oct  6 14:00 pgdg.sources
# утилиты
    # просмотр установленных утилит
    dpkg -L postgresql-common
# утилита просмотра кластеров
pg_lsclusters
    # результат выполнения
    Ver Cluster Port Status Owner Data directory Log file
    #Версия/Имя кластера/Порт/Статус/Владелец/Каталог с данными/ Лог файл


# установка конкретной версии postgresql
    # устанавливаем бинарники
    sudo apt install -y postgresql-17
       # проверка результата
       dpkg -l | grep postgresql-17
       # результат выполнения
       ii  postgresql-17                              17.11-1.pgdg26.04+2                        amd64        The World's Most Advanced Open Source Relational Database
```
```sh   
# установка кластера
sudo pg_createcluster 17 pg_17 --port 5434
    # результат выполнения команды
    VVer Cluster Port Status Owner    Data directory               Log file
    17  pg_17   5434 down   postgres /var/lib/postgresql/17/pg_17 /var/log/postgresql/postgresql-17-pg_17.log

        # просмотр кластера
        pg_lsclusters
            # результат
            Ver Cluster Port Status Owner    Data directory               Log file
            17  main    5432 online postgres /var/lib/postgresql/17/main  /var/log/postgresql/postgresql-17-main.log
            17  pg_17   5434 down   postgres /var/lib/postgresql/17/pg_17 /var/log/postgresql/postgresql-17-pg_17.log

            # main не нужен, создан автоматически
            sudo pg_ctlcluster 17 main stop
            sudo pg_dropcluster 17 main
                # результат
                pg_lsclusters
                    Ver Cluster Port Status Owner    Data directory               Log file
                    7  pg_17   5434 down   postgres /var/lib/postgresql/17/pg_17 /var/log/postgresql/postgresql-17-pg_17.log

# просмотр файлов конфигурации
cd /etc/postgresql/17/pg_17
ls -la
    # результат
    conf.d  environment  pg_ctl.conf  pg_hba.conf  pg_ident.conf  postgresql.conf  start.conf
#  данные
cd /var/lib/postgresql/17/pg_17
ls
    # результат
    PG_VERSION  pg_commit_ts  pg_multixact  pg_serial     pg_stat_tmp  pg_twophase  postgresql.auto.conf
    base        pg_dynshmem   pg_notify     pg_snapshots  pg_subtrans  pg_wal
    global      pg_logical    pg_replslot   pg_stat       pg_tblspc    pg_xact
# логи
cd /var/log/postgresql/
ls
    # результат
    postgresql-17-pg_-_17.log
# исполняемые файлы
cd /usr/lib/postgresql/17/bin
ls
    # результат
    clusterdb   oid2name           pg_config            pg_isready      pg_test_fsync    pgbench
    createdb    pg_amcheck         pg_controldata       pg_receivewal   pg_test_timing   postgres
    createuser  pg_archivecleanup  pg_createsubscriber  pg_recvlogical  pg_upgrade       psql
    dropdb      pg_basebackup      pg_ctl               pg_resetwal     pg_verifybackup  reindexdb
    dropuser    pg_checksums       pg_dump              pg_restore      pg_waldump       vacuumdb
    initdb      pg_combinebackup   pg_dumpall           pg_rewind       pg_walsummary    vacuumlo

# Проверка запущенных процессов с postgres
ps -auxf | grep postgres
ps -ef | grep postgres

# Команда запуска кластера
sudo pg_ctlcluster start 17 pg_17
    pg_lsclusters
    # Результат
    Ver Cluster Port Status Owner    Data directory               Log file
    17  pg_17   5434 online postgres /var/lib/postgresql/17/pg_17 /var/log/postgresql/postgresql-17-pg_17.log
```
```sh
# подключение к postgresql
sudo -u postgres psql -p 5434
    # результат
    psql (17.11 (Ubuntu 17.11-1.pgdg26.04+2))
    Type "help" for help. 
    postgres=#
# просмотр параметров подключения    
postgres=# \conninfo
    # результат
    You are connected to database "postgres" as user "postgres" via socket in "/var/run/postgresql" at port "5434".
# задать пароль пользователю postgres
\password
# изменения в файлах конфигурации
show hba_file;
    # результат  
                   hba_file
    --------------------------------------
    /etc/postgresql/17/pg_17/pg_hba.conf # кто с каких ip, к какой бд может подключаться
    (1 row)

show config_file; # настройки кластера
    # результат 
                   config_file
    ------------------------------------------
    /etc/postgresql/17/pg_17/postgresql.conf
    (1 row)
show data_directory # файлы базы данных

select current_setting('data_directory'); # вызов встроенной функции PostgreSQL, которая возвращает значение текущего параметра конфигурации data_directory
    # результат 
           current_setting
    ------------------------------
    /var/lib/postgresql/17/pg_17
    (1 row)



# редактирование файлов конфигурации
sudo vi /etc/postgresql/17/pg_17/pg_hba.conf

# Database administrative login by Unix domain socket
local   all             postgres                                peer  # строка говорит о том что локально по соккету может подключиться кто угодно.
host    replication     all             127.0.0.1/32            scram-sha-256 # подключение по сети, для яндекс облака тоже надо использовать
# в варианте ниже можно подключиться к кластеру с любого ip
host    replication     all             0.0.0.0/0            scram-sha-256 # 

sudo vi /etc/postgresql/17/pg_17/postgresql.conf
    # срока listen_addresses = 'localhost'          # what IP address(es) to listen on;
    listen_addresses = '*' # принимать соединения со всех адресов
 
# подключение к postgresql с указанием хоста
sudo -u postgres psql -p 5434 -h localhost
    # проверяем подключение
        postgres=# \conninfo
        You are connected to database "postgres" as user "postgres" on host "localhost" (address "127.0.0.1") at port "5434".

```
# Работаем с кластером
```sh
# так как менялись настройки, выполняем рестарт кластера
sudo pg_ctlcluster 17 pg_17 restart
# просмотр статуса
sudo pg_ctlcluster 17 pg_17 reboot
# перечитать параметры без остановки кластера
sudo pg_ctlcluster 17 pg_17 status
# Примечание. На одном экземпляре - одна производственная база данных (общий WAL, настройки, ресурсы памяти)
# Создание нового экземпляра
sudo pg_createcluster 18 instance02 --port 5435
sudo pg_ctlcluster 18 instance02 start
# Удаление экземпляра
sudo pg_ctlcluster 18 instance02 start
sudo pg_dropcluster 18 instance02 stop
#
ps auxf | grep postgres
```
# Подключение к postgres DBeaver
```sh
# создание бд
create database test;
\с # подключение к бд
create table test1 (trest int, name text)
```
```sh
# установка Docker
# Add Docker's official GPG key:
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# проверка установки 
docker ps
    # результат
    CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
# создание сети docker с именем pg-net
sudo docker network create pg-net
    # результат
    4d9cd0ef3fedb0d8887de819b032f2de503efe842465c359d83ebd90d448a565

sudo docker run --name pg-docker \
  --network pg-net \
  -e POSTGRES_PASSWORD=postgres \
  -d \
  -p 5432:5432 \
  -v /var/lib/postgresql/docker-pg17:/var/lib/postgresql/data \
  postgres:17
  
   # Синтаксис sudo docker run [опции] <образ> [команда]
   # docker run - создать новый контейнер create + start в одной команде
   # опции
       # --name pg-docker имя контейнера
       # --network pg-net
       # -e задаёт переменную окружения внутри контейнера
       # -d запустить контейнер в фоне
       # POSTGRES_PASSWORD=postgres переменная задает пароль суперпользователя
       # -p 5432:5432 проброс порта с хоста в контейнер, первый на хосте второй в контейнере.
           # следует отметить что системный PG слушает 5434, контейнерный 5432 
       # -v /var/lib/postgresql/docker-pg17:/var/lib/postgresql/data - монтирование папки Хост:контейнер
           # var/lib/postgresql/docker-pg17 - папка на хосте
           # /var/lib/postgresql/data - папка внутри контейнера
           # 
   # postgres:17 - имя образа
docker ps
    # результат
    CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS      NAMES
    a1b56b45d088   postgres:18   "docker-entrypoint.s…"   5 minutes ago   Up 5 minutes   5432/tcp   festive_mayer
# остановка
sudo docker stop a1b56b45d088
# просмотр контейнеров с оснановленными
sudo docker ps -a
# удаление
sudo docker rm ID контейнера

docker pull postgres:17 


```sh

docker pull postgres:18
# просмотр образов, которые хранятся на машине
sudo docker image ls
# По версиям хостовой 17 контейнерный 18, возможны конфликты? По видимому нет.
# создание контейнера

docker run --rm -d -e POSTGRES_PASSWORD=postgres postgres:17

    # --rm удаляет контейнер после выхода из него;

    # -d  работать в фоне

    # -e  создание переменной окружения POSTGRES_PASSWORD=postgres

    # просмотр

docker pc
    # результат
sudo docker run --name postgres:18
sudo docker run --rm -d \
    --name postgres18 \
    -e POSTGRES_PASSWORD=postgres \
    -p 5432:5432 \
    -v /var/lib/postgresql/docker-pg18:/var/lib/postgresql/data \
    postgres:18

sudo docker run --rm -d --name postgres18 -e POSTGRES_PASSWORD=postgres -p 5432:5432 -v /var/lib/postgresql/docker-pg18:/var/lib/postgresql/data  postgres:18

# ошибка нет каталога /var/lib/postgresql/docker-pg18
sudo mkdir /var/lib/postgresql/docker-pg18
    drwx------ 19 dnsmasq  root     4096 Oct  7 16:25 docker-pg17
    drwxr-xr-x  2 root     root     4096 Oct  8 18:50 docker-pg18
sudo chown -R 999:root /var/lib/postgresql/docker-pg18
sudo chmod 700 /var/lib/postgresql/docker-pg18
   drwx------ 19 dnsmasq  root     4096 Oct  7 16:25 docker-pg17
   drwx------  2 dnsmasq  root     4096 Oct  8 18:50 docker-pg18


# ошибка все равно
# меняем на 17 версию - всё пучком
gorskiy-nn@gorskiy-nn-VirtualBox:/var/lib/postgresql$ sudo docker run -d --name postgres17 -e POSTGRES_PASSWORD=postgres -p 5432:5432 -v /var/lib/postgresql/docker-pg17:/var/lib/postgresql/data  postgres:17
    a3eac002aff3a0ac6ec38d42c6d69ca243b6bde283d09599b5839ab7b4cf4258
    sudo docker ps
    CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                         NAMES
    a3eac002aff3   postgres:17   "docker-entrypoint.s…"   3 seconds ago   Up 2 seconds   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp   postgres17
        sudo docker stop a3eac002aff3
        sudo docker rm a3eac002aff3


# повторная попытка создать контейнер без data в :/var/lib/postgresql 
sudo docker run --rm -d --name postgres18 -e POSTGRES_PASSWORD=postgres -p 5432:5432 -v /var/lib/postgresql/docker-pg18:/var/lib/postgresql  postgres:18
    c59928a7dc66059c0442a076d801c6d8754232238b67b000f292da325cfd4962
sudo docker ps
    CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                         NAMES
    c59928a7dc66   postgres:18   "docker-entrypoint.s…"   6 seconds ago   Up 5 seconds   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp   postgres18


   

# проверить подключение к postgres из Beaver
# не подключается 
gorskiy-nn@gorskiy-nn-VirtualBox:/var/lib/postgres$ telnet localhost 5432
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.

 

# Подключение к контейнеру из командной строки

 

docker exec -it \
-e POSTGRES_PASSWORD=postgres \
postgres18 \
psql -U postgres

    # exec -it
    # -U postgres подключение пользавателем postgres
# работает
# подключение из windows не работает, хотя порт слушается
# подключение с другой виртуалки - успех
psql -h 10.62.11.82 -p 5432 -U postgres -d postgres

    gorskiy-nn@gorskiy-nn-VirtualBox:~$ psql -h 10.62.11.82 -p 5432 -U postgres -d postgres
    Password for user postgres:
    psql (17.11 (Ubuntu 17.11-1.pgdg26.04+2), server 18.6 (Debian 18.6-1.pgdg13+2))
    WARNING: psql major version 17, server major version 18.
         Some psql features might not work.
    Type "help" for help.


create database test2;

\c test2;

create table test (id int);
insert into test values (1);

postgres=# create database test2;
CREATE DATABASE
postgres=# \c test2;
psql (17.11 (Ubuntu 17.11-1.pgdg26.04+2), server 18.6 (Debian 18.6-1.pgdg13+2))
WARNING: psql major version 17, server major version 18.
         Some psql features might not work.
You are now connected to database "test2" as user "postgres".
test2=# create table test (id int);
insert into test values (1);
CREATE TABLE
INSERT 0 1
test2=# select * from test
test2-# ;
 id
----
  1
(1 row)

```
```sh 
# проверить подключение к postgres из Beaver на потом
# остановка контейнера
sudo docker stop postgres18
   # результат
   #gorskiy-nn@gorskiy-nn-VirtualBox:/var/lib/postgres$ sudo docker stop postgres18
   #postgres18
   sudo docker ps
      CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAME
#  создание контейнера повторно
   sudo docker run --rm -d --name postgres18 -e POSTGRES_PASSWORD=postgres -p 5432:5432 -v /var/lib/postgresql/docker-pg18:/var/lib/postgresql  postgres:18
   # результат
      gorskiy-nn@gorskiy-nn-VirtualBox:/var/lib/postgres$ sudo docker run --rm -d --name postgres18 -e POSTGRES_PASSWORD=postgres -p 5432:5432 -v /var/lib/postgresql/docker-pg18:/var/lib/postgresql  postgres:18
      a4c275e8c1c5c505a01bc75191533e993466322ed799392f8d91e4c8b8d149f2
    gorskiy-nn@gorskiy-nn-VirtualBox:/var/lib/postgres$ sudo docker ps
    CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                         NAMES
    a4c275e8c1c5   postgres:18   "docker-entrypoint.s…"   19 seconds ago   Up 18 seconds   0.0.0.0:5432->5432/tcp, [::]:5432->5432/tcp   postgres18

 # проверка базы с другого сервака
 gorskiy-nn@gorskiy-nn-VirtualBox:~$ psql -h 10.62.11.82 -p 5432 -U postgres -d postgres
    #Password for user postgres:
    #psql (17.11 (Ubuntu 17.11-1.pgdg26.04+2), server 18.6 (Debian 18.6-1.pgdg13+2))
    #WARNING: psql major version 17, server major version 18.
    #Some psql features might not work.
    #Type "help" for help.

postgres=# /c test2
#postgres-# select * from test;
#ERROR:  syntax error at or near "/"
#LINE 1: /c test2
        ^
postgres=# \c test2
psql (17.11 (Ubuntu 17.11-1.pgdg26.04+2), server 18.6 (Debian 18.6-1.pgdg13+2))
WARNING: psql major version 17, server major version 18.
         Some psql features might not work.
You are now connected to database "test2" as user "postgres".
test2=# select * from test;
 id
----
  1
(1 row)


! [](file_name.png)














