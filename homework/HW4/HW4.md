Pgbench: базовый тюнинг

Цель:
повысить производительность PostgreSQL на тестовом стенде для быстрой оценки эффекта тюнинга конфигурации под нагрузкой;


Описание/Пошаговая инструкция выполнения домашнего задания:
Контекст

Перед пилотным запуском сервиса требуется быстро оценить, какой прирост производительности можно получить «малой кровью» на текущем стенде; времени на глубокую оптимизацию нет, нужен воспроизводимый результат: baseline → изменения → повторный тест → вывод;


Описание / пошаговая инструкция выполнения ДЗ

Используйте существующую ВМ/стенд с PostgreSQL;
- выполнено
Подготовьте pgbench: инициализируйте тестовую базу (scale выбрать 10);
- выполнено
Выполните baseline-прогон pgbench с фиксированными настройками (например, 60 секунд, 50 клиентов, 10 потока);
- выполнено
Зафиксируйте tps baseline и условия теста (scale, время, клиенты/потоки);
- выполнено
Измените параметры PostgreSQL для получения максимальной производительности
- выполнено
Перезапустите кластер (если требуется) и повторите pgbench с теми же настройками;
- выполнено
Сравните tps до/после и сделайте вывод, какие изменения дали эффект;
- выполнено
Кратко обоснуйте изменение каких параметров больше всего влияет на повышение производительности;
- выполнено

Используйте существующую ВМ/стенд с PostgreSQL;
    #собираем с нуля
```sh
sudo apt update
sudo apt -y install vim
    sudo apt install openssh-server
        sudo apt install postgresql-common
            sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
                sudo apt install postgresql-18
                    sudo pg_dropcluster 18 main -stop
                        sudo pg_createcluster 18 pg_18 --port 5859
                            pg_lsclusters
                                sudo pg_ctlcluster 18 pg_18 start
                                    sudo -u postgres psql --port 5859
                                        \password
                                    sudo -u postgres psql --port 5859 -h localhost
                                    sudo su postgres
                                        psql
                                        show hba_file;
                                            sudo vi /etc/postgresql/18/pg_18/pg_hba.conf
                                                # IPv4 local connections:
                                                host    all             all             10.62.11.102/32            scram-sha-256
                                                host    all             all             10.62.11.82/32            scram-sha-256
                                                    show config_file;
                                                        sudo vi /etc/postgresql/18/pg_18/postgresql.conf
                                                            listen_addresses = '*'
                                                                sudo pg_ctlcluster 18 pg_18 stop
                                                                sudo pg_ctlcluster 18 pg_18 start
                                    sudo -u postgres psql --port 5859 -h 10.62.11.95
```

```sql

select * from pg_settings;
    select * from pg_settings where name='max_connections'; 
    show max_connections   
        select * from pg_settings where name = 'work_mem';
        show work_mem
        show hba_file;
select * from pg_settings
where name='max_connections';
select * from pg_settings where name = 'work_mem';
        show work_mem
        show max_connections
select * from pg_conf_file;
select source, count(*) from pg_settings group by source order by 2 desc;
select context, count(*) from pg_settings group by context order by 2 desc;
select category, count(*) from pg_settings group by category order by 2 desc;
select setting||' x '||coalesce(unit,'units') from pg_settings where name='max_connections';
        show max_connections;
select current_setting('max_connections');
    alter system set max_connections = '200';
SELECT name, setting, pending_restart FROM pg_settings WHERE name = 'max_connections';
SELECT pg_reload_conf();
select sourcefile, count(*) from pg_file_settings group by sourcefile;
select * from pg_file_settings where name = 'work_mem';
            

```
Подготовьте pgbench: инициализируйте тестовую базу (scale выбрать 10);

```sql
CREATE DATABASE OTUS;
    ALTER SYSTEM SET
 max_connections = '97';
ALTER SYSTEM SET
 shared_buffers = '2560MB';
ALTER SYSTEM SET
 effective_cache_size = '7680MB';
ALTER SYSTEM SET
 maintenance_work_mem = '640MB';
ALTER SYSTEM SET
 checkpoint_completion_target = '0.9';
ALTER SYSTEM SET
 wal_buffers = '16MB';
ALTER SYSTEM SET
 default_statistics_target = '100';
ALTER SYSTEM SET
 random_page_cost = '4';
ALTER SYSTEM SET
 effective_io_concurrency = '2';
ALTER SYSTEM SET
 work_mem = '9008kB';
ALTER SYSTEM SET
 huge_pages = 'off';
ALTER SYSTEM SET
 min_wal_size = '1GB';
ALTER SYSTEM SET
 max_wal_size = '4GB';
ALTER SYSTEM SET
 max_worker_processes = '5';
ALTER SYSTEM SET
 max_parallel_workers_per_gather = '3';
ALTER SYSTEM SET
 max_parallel_workers = '5';
ALTER SYSTEM SET
 max_parallel_maintenance_workers = '3';

     select * from pg_file_settings where sourcefile ='/var/lib/postgresql/18/pg_18/postgresql.auto.conf';
"sourcefile"	"sourceline"	"seqno"	"name"	"setting"	"applied"	"error"
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	3	27	"max_connections"	"97"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	4	28	"shared_buffers"	"2560MB"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	5	29	"effective_cache_size"	"7680MB"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	6	30	"maintenance_work_mem"	"640MB"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	7	31	"checkpoint_completion_target"	"0.9"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	8	32	"wal_buffers"	"16MB"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	9	33	"default_statistics_target"	"100"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	10	34	"random_page_cost"	"4"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	11	35	"effective_io_concurrency"	"2"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	12	36	"work_mem"	"9008kB"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	13	37	"huge_pages"	"off"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	14	38	"min_wal_size"	"1GB"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	15	39	"max_wal_size"	"4GB"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	16	40	"max_worker_processes"	"5"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	17	41	"max_parallel_workers_per_gather"	"3"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	18	42	"max_parallel_workers"	"5"	true	
"/var/lib/postgresql/18/pg_18/postgresql.auto.conf"	19	43	"max_parallel_maintenance_workers"	"3"	true	


SELECT name, setting, pending_restart FROM pg_settings WHERE name IN ('max_connections','shared_buffers','effective_cache_size','work_mem');
"name"	"setting"	"pending_restart"
"effective_cache_size"	"983040"	false
"max_connections"	"97"	false
"shared_buffers"	"327680"	false
"work_mem"	"9008"	false

```
```sh
pgbench -i otus
        dropping old tables...
         NOTICE:  table "pgbench_accounts" does not exist, skipping
         NOTICE:  table "pgbench_branches" does not exist, skipping
         NOTICE:  table "pgbench_history" does not exist, skipping
         NOTICE:  table "pgbench_tellers" does not exist, skipping
         creating tables...
         generating data (client-side)...
         vacuuming...
         creating primary keys...
         done in 0.19 s (drop tables 0.00 s, create tables 0.02 s, client-side generate 0.09 s, vacuum 0.04 s, primary keys 0.04 s).
pgbench -i -s 10 otus
    dropping old tables...
    creating tables...
    generating data (client-side)...
    vacuuming...
    creating primary keys...
    done in 4.59 s (drop tables 0.02 s, create tables 0.01 s, client-side generate 3.35 s, vacuum 0.07 s, primary keys 1.14 s).

postgres=# \c otus
You are now connected to database "otus" as user "postgres".
otus=# \dt
                List of tables
 Schema |       Name       | Type  |  Owner
--------+------------------+-------+----------
 public | pgbench_accounts | table | postgres
 public | pgbench_branches | table | postgres
 public | pgbench_history  | table | postgres
 public | pgbench_tellers  | table | postgres
(4 rows)

```
```sh

 postgres@gorskiy-nn-VirtualBox:/home/gorskiy-nn$ pgbench -c 50 -j 6 -P 10 -T 60 -d otus
pgbench (18.6 (Ubuntu 18.6-1.pgdg26.04+2))
starting vacuum...end.
progress: 10.0 s, 2833.7 tps, lat 17.558 ms stddev 22.046, 0 failed
progress: 20.0 s, 2017.7 tps, lat 24.820 ms stddev 35.534, 0 failed
progress: 30.0 s, 2160.0 tps, lat 23.123 ms stddev 27.852, 0 failed
progress: 40.0 s, 2168.7 tps, lat 23.077 ms stddev 28.878, 0 failed
progress: 50.0 s, 1894.3 tps, lat 26.361 ms stddev 33.023, 0 failed
progress: 60.0 s, 1824.4 tps, lat 27.406 ms stddev 32.937, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 50
number of threads: 6
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 129040
number of failed transactions: 0 (0.000%)
latency average = 23.248 ms
latency stddev = 29.994 ms
initial connection time = 18.764 ms
tps = 2148.993639 (without initial connection time)

# отключаем fsync и synchronous_commit
ALTER SYSTEM SET synchronous_commit = 'off';
SELECT pg_reload_conf();
SHOW synchronous_commit;
    off
ALTER SYSTEM SET fsync = 'off';
#рестарт кластера
SHOW fsync;
    off
pgbench -i -s 10 otus
pgbench -c 50 -j 6 -P 10 -T 60 -d otus
postgres@gorskiy-nn-VirtualBox:/home/gorskiy-nn$ pgbench -c 50 -j 6 -P 10 -T 60 -d otus
pgbench (18.6 (Ubuntu 18.6-1.pgdg26.04+2))
starting vacuum...end.
progress: 10.0 s, 25736.8 tps, lat 1.938 ms stddev 1.414, 0 failed
progress: 20.0 s, 25675.6 tps, lat 1.947 ms stddev 1.523, 0 failed
progress: 30.0 s, 26145.0 tps, lat 1.911 ms stddev 1.465, 0 failed
progress: 40.0 s, 25532.2 tps, lat 1.957 ms stddev 1.616, 0 failed
progress: 50.0 s, 24494.0 tps, lat 2.040 ms stddev 1.709, 0 failed
progress: 60.0 s, 24518.6 tps, lat 2.038 ms stddev 1.746, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 50
number of threads: 6
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 1521069
number of failed transactions: 0 (0.000%)
latency average = 1.971 ms
latency stddev = 1.585 ms
initial connection time = 21.239 ms
tps = 25345.680825 (without initial connection time)


tps = 2148 на 6 ядрах с включенным fsync и synchronous_commit
tps = 25345 на 6 ядрах с отключенным fsync и synchronous_commit
# с отключенным fsync и synchronous_commit видим максимальную производительность. Рост в 12 раз
# на двух ядрах тот же рост в 12 раз
tps = 447.68 на 2 ядрах с включенным fsync и synchronous_commit
tps = 5574.02 на 2 ядрах с отключенным fsync и synchronous_commit
# отключение параметров fsync и synchronous_commit больше всего влияет на производитеьность
```