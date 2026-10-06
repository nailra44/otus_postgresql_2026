Установка_postgresql

##TODO:
- [] Создайте ВМ с Ubuntu 22.04/24.04 или подготовьте хост, на котором будет развёрнут Docker;

- выполнено

TODO: Установите Docker Engine;
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














