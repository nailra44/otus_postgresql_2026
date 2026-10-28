Домашнее задание
Журналы и контрольные точки

Цель:
оценить влияние настроек на объём генерируемых WAL файлов и производительность записи под нагрузкой;


Описание/Пошаговая инструкция выполнения домашнего задания:
Контекст

Перед запуском распродажи «секундных» билетов дежурный DBA замечает: каждые полминуты база кратко замедляется, реплика отстаёт, а резервные копии не укладываются в окно. Поддержка получает жалобы «платёж прошёл — билета нет», и нужно понять, что происходит при записи под нагрузкой. Руководитель продукта требует максимальной скорости, финансовый контролёр — минимального риска потери последних операций при аварии. Требуется измерить объём генерируемых изменений, связать его с регулярным сбросом на диск, сравнить два режима фиксации и предложить обоснованные настройки — что выбрать и почему?


Описание / пошаговая инструкция выполнения ДЗ

Используйте стенд с PostgreSQL 18  и установите pgbench;
- выполнено
Настройте выполнение контрольной точки раз в 30 секунд (параметр времени) и включите логирование контрольных точек;
- выполнено
Зафиксируйте начальные значения статистики контрольных точек и WAL (снимок «до»);
- выполнено
Запустите нагрузку pgbench на 10 минут и зафиксируйте TPS;
- выполнено
Зафиксируйте конечные значения статистики контрольных точек и WAL (снимок «после»);
- выполнено
Рассчитайте объём WAL за 10 минут и средний объём WAL на одну контрольную точку;
- выполнено
Проверьте по статистике/логам, выполнялись ли контрольные точки строго по расписанию, и объясните отклонения;
- выполнено
Сравните TPS при синхронном и асинхронном подтверждении коммита (2 прогона pgbench в сопоставимых условиях) и объясните разницу;
- выполнено

Используйте стенд с PostgreSQL 18  и установите pgbench;
```sh
#pg_lsclusters
    #Ver Cluster Port Status Owner    Data directory               Log file
    18  pg_18   5858 online postgres /var/lib/postgresql/18/pg_18 /var/log/postgresql/postgresql-18-pg_18.log
        pgbench --version
            pgbench (PostgreSQL) 18.6 (Ubuntu 18.6-1.pgdg26.04+2)
```
Настройте выполнение контрольной точки раз в 30 секунд (параметр времени) и включите логирование контрольных точек;
```sh
#Установка интервала 30 секунд
ALTER SYSTEM SET checkpoint_timeout = '30s';
    #Включение логирования контрольных точек
    ALTER SYSTEM SET log_checkpoints = 'on';
        #Проверка включения параметров 
            SELECT sourcefile, name, setting, applied, error
            FROM pg_file_settings 
            WHERE sourcefile LIKE '%postgresql.auto.conf';
                    sourcefile                     | sourceline | seqno |        name        | setting | applied | error
---------------------------------------------------+------------+-------+--------------------+---------+---------+-------
 /var/lib/postgresql/18/pg_18/postgresql.auto.conf |          3 |    27 | checkpoint_timeout | 30s     | t       |
 /var/lib/postgresql/18/pg_18/postgresql.auto.conf |          4 |    28 | log_checkpoints    | on      | t       |
(2 rows)
            #Перечитываем конфиг
            SELECT pg_reload_conf();
			    pg_reload_conf
                ----------------
                 t
                (1 row)
```
Зафиксируйте начальные значения статистики контрольных точек и WAL (снимок «до»);
```sh
#статистика работы процесса checkpointer
SELECT * FROM pg_stat_checkpointer;
 num_timed | num_requested | num_done | restartpoints_timed | restartpoints_req | restartpoints_done | write_time | sync_time | buffers_written | slru_written |          stats_reset
-----------+---------------+----------+---------------------+-------------------+--------------------+------------+-----------+-----------------+--------------+-------------------------------
        82 |             1 |        3 |                   0 |                 0 |                  0 |       2019 |       769 |           16000 |            3 | 2026-10-26 19:41:49.455492+03
(1 row)
#накопленные счётчики генерации и записи WAL
SELECT * FROM pg_stat_wal;
 wal_records | wal_fpi | wal_bytes | wal_buffers_full |          stats_reset
-------------+---------+-----------+------------------+-------------------------------
          15 |       3 |      7959 |                0 | 2026-10-26 19:41:49.455492+03
 SELECT backend_type, object, context, writes, write_bytes, write_time
FROM pg_stat_io
WHERE object = 'wal';
    backend_type     | object | context | writes | write_bytes | write_time
---------------------+--------+---------+--------+-------------+------------
 client backend      | wal    | init    |      0 |           0 |          0
 client backend      | wal    | normal  |      0 |           0 |          0
 autovacuum launcher | wal    | init    |      0 |           0 |          0
 autovacuum launcher | wal    | normal  |      0 |           0 |          0
 autovacuum worker   | wal    | init    |      0 |           0 |          0
 autovacuum worker   | wal    | normal  |      0 |           0 |          0
 background worker   | wal    | init    |      0 |           0 |          0
 background worker   | wal    | normal  |      0 |           0 |          0
 walsender           | wal    | init    |      0 |           0 |          0
 walsender           | wal    | normal  |      0 |           0 |          0
 slotsync worker     | wal    | init    |      0 |           0 |          0
 slotsync worker     | wal    | normal  |      0 |           0 |          0
 standalone backend  | wal    | init    |      0 |           0 |          0
 standalone backend  | wal    | normal  |      0 |           0 |          0
 background writer   | wal    | init    |      0 |           0 |          0
 background writer   | wal    | normal  |      0 |           0 |          0
 checkpointer        | wal    | init    |      0 |           0 |          0
 checkpointer        | wal    | normal  |      3 |       24576 |          0
 io worker           | wal    | init    |      0 |           0 |          0
 io worker           | wal    | normal  |      0 |           0 |          0
 startup             | wal    | init    |      0 |           0 |          0
 startup             | wal    | normal  |      0 |           0 |          0
 walreceiver         | wal    | init    |      0 |           0 |          0
 walreceiver         | wal    | normal  |      0 |           0 |          0
 walsummarizer       | wal    | init    |      0 |           0 |          0
 walsummarizer       | wal    | normal  |      0 |           0 |          0
 walwriter           | wal    | init    |      0 |           0 |          0
 walwriter           | wal    | normal  |      8 |       65536 |          0
(28 rows)

```
Запустите нагрузку pgbench на 10 минут и зафиксируйте TPS;
```sh
#Инициализация БД
su postgres
postgres@gorskiy-nn-VirtualBox:/home/gorskiy-nn$ pgbench -i otushw6
dropping old tables...
creating tables...
generating data (client-side)...
vacuuming...
creating primary keys...
done in 0.69 s (drop tables 0.26 s, create tables 0.11 s, client-side generate 0.19 s, vacuum 0.09 s, primary keys 0.05 s).
postgres@gorskiy-nn-VirtualBox:/home/gorskiy-nn$
    #запуск нагрузки
postgres@gorskiy-nn-VirtualBox:/home/gorskiy-nn$ pgbench -c 8 -j 4 -T 600 -P 60 -U postgres otushw6
pgbench (18.6 (Ubuntu 18.6-1.pgdg26.04+2))
starting vacuum...end.
progress: 60.0 s, 380.2 tps, lat 21.035 ms stddev 16.034, 0 failed
progress: 120.0 s, 368.8 tps, lat 21.695 ms stddev 16.388, 0 failed
progress: 180.0 s, 382.4 tps, lat 20.917 ms stddev 16.577, 0 failed
progress: 240.0 s, 442.1 tps, lat 18.096 ms stddev 14.871, 0 failed
progress: 300.0 s, 416.7 tps, lat 19.195 ms stddev 15.999, 0 failed
progress: 360.0 s, 365.9 tps, lat 21.865 ms stddev 16.116, 0 failed
progress: 420.0 s, 407.6 tps, lat 19.622 ms stddev 15.080, 0 failed
progress: 480.0 s, 414.8 tps, lat 19.291 ms stddev 14.693, 0 failed
progress: 540.0 s, 347.3 tps, lat 23.033 ms stddev 16.176, 0 failed
progress: 600.0 s, 389.0 tps, lat 20.568 ms stddev 15.695, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 234888
number of failed transactions: 0 (0.000%)
latency average = 20.434 ms
latency stddev = 15.806 ms
initial connection time = 9.945 ms
tps = 391.473101 (without initial connection time)




```
Зафиксируйте конечные значения статистики контрольных точек и WAL (снимок «после»);
```sh
#статистика работы процесса checkpointer
SELECT * FROM pg_stat_checkpointer;
num_timed | num_requested | num_done | restartpoints_timed | restartpoints_req | restartpoints_done | write_time | sync_time | buffers_written | slru_written |          stats_reset
-----------+---------------+----------+---------------------+-------------------+--------------------+------------+-----------+-----------------+--------------+-------------------------------
       143 |             1 |       24 |                   0 |                 0 |                  0 |     566166 |      1560 |           56018 |           67 | 2026-10-26 19:41:49.455492+03
(1 row)
#накопленные счётчики генерации и записи WAL
SELECT * FROM pg_stat_wal;
postgres=# SELECT * FROM pg_stat_wal;
 wal_records | wal_fpi | wal_bytes | wal_buffers_full |          stats_reset
-------------+---------+-----------+------------------+-------------------------------
     1778172 |   41263 | 446050369 |              681 | 2026-10-26 19:41:49.455492+03
(1 row)
postgres=# SELECT backend_type, object, context, writes, write_bytes, write_time
FROM pg_stat_io
WHERE object = 'wal';
    backend_type     | object | context | writes | write_bytes | write_time
---------------------+--------+---------+--------+-------------+------------
 client backend      | wal    | init    |      0 |           0 |          0
 client backend      | wal    | normal  | 235686 |  2369536000 |          0
 autovacuum launcher | wal    | init    |      0 |           0 |          0
 autovacuum launcher | wal    | normal  |      0 |           0 |          0
 autovacuum worker   | wal    | init    |      0 |           0 |          0
 autovacuum worker   | wal    | normal  |     53 |     1196032 |          0
 background worker   | wal    | init    |      0 |           0 |          0
 background worker   | wal    | normal  |      0 |           0 |          0
 walsender           | wal    | init    |      0 |           0 |          0
 walsender           | wal    | normal  |      0 |           0 |          0
 slotsync worker     | wal    | init    |      0 |           0 |          0
 slotsync worker     | wal    | normal  |      0 |           0 |          0
 standalone backend  | wal    | init    |      0 |           0 |          0
 standalone backend  | wal    | normal  |      0 |           0 |          0
 background writer   | wal    | init    |      0 |           0 |          0
 background writer   | wal    | normal  |      0 |           0 |          0
 checkpointer        | wal    | init    |      0 |           0 |          0
 checkpointer        | wal    | normal  |     50 |      434176 |          0
 io worker           | wal    | init    |      0 |           0 |          0
 io worker           | wal    | normal  |      0 |           0 |          0
 startup             | wal    | init    |      0 |           0 |          0
 startup             | wal    | normal  |      0 |           0 |          0
 walreceiver         | wal    | init    |      0 |           0 |          0
 walreceiver         | wal    | normal  |      0 |           0 |          0
 walsummarizer       | wal    | init    |      0 |           0 |          0
 walsummarizer       | wal    | normal  |      0 |           0 |          0
 walwriter           | wal    | init    |      0 |           0 |          0
 walwriter           | wal    | normal  |    243 |     3342336 |          0
(28 rows)


```
Рассчитайте объём WAL за 10 минут и средний объём WAL на одну контрольную точку;

```sh

WAL за 10 минут = wal_bytes(после) − wal_bytes(до)
446050369 − 7959 = 446042410 байт. 425 МБ
Средний WAL на контрольную точку WAL за 10 минут / на контрольные точки
Средний WAL на контрольную точку = 446042410 / 22 = 20274655 байт = 19.3 МБ

```
Проверьте по статистике/логам, выполнялись ли контрольные точки строго по расписанию, и объясните отклонения;
```sh
postgres=# SHOW logging_collector;
 logging_collector
-------------------
 off
(1 row)
#лог не пишется, начинаем сначала
#сброс статистики
SELECT pg_stat_reset_shared();	


#Инициализация БД
   

su postgres
postgres@gorskiy-nn-VirtualBox:/home/gorskiy-nn$ pgbench -i -s 10 -U postgres otushw6
dropping old tables...
creating tables...
generating data (client-side)...
vacuuming...
creating primary keys...
done in 2.95 s (drop tables 0.85 s, create tables 0.07 s, client-side generate 1.55 s, vacuum 0.12 s, primary keys 0.36 s).

    #запуск нагрузки
postgres@gorskiy-nn-VirtualBox:/home/gorskiy-nn$ pgbench -c 8 -j 4 -T 600 -P 60 -U postgres otushw6
pgbench (18.6 (Ubuntu 18.6-1.pgdg26.04+2))
starting vacuum...end.
progress: 60.0 s, 939.0 tps, lat 8.516 ms stddev 18.581, 0 failed
progress: 120.0 s, 931.7 tps, lat 8.585 ms stddev 8.875, 0 failed
progress: 180.0 s, 949.9 tps, lat 8.421 ms stddev 10.113, 0 failed
progress: 240.0 s, 901.2 tps, lat 8.872 ms stddev 10.715, 0 failed
progress: 300.0 s, 957.4 tps, lat 8.360 ms stddev 8.288, 0 failed
progress: 360.0 s, 911.7 tps, lat 8.761 ms stddev 9.867, 0 failed
progress: 420.0 s, 951.2 tps, lat 8.422 ms stddev 8.609, 0 failed
progress: 480.0 s, 936.1 tps, lat 8.545 ms stddev 9.492, 0 failed
progress: 540.0 s, 918.0 tps, lat 8.714 ms stddev 9.915, 0 failed
progress: 600.0 s, 929.6 tps, lat 8.604 ms stddev 8.515, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 559544
number of failed transactions: 0 (0.000%)
latency average = 8.577 ms
latency stddev = 10.689 ms
initial connection time = 16.543 ms
tps = 932.575798 (without initial connection time)

#статистика работы процесса checkpointer
SELECT * FROM pg_stat_checkpointer;
num_timed | num_requested | num_done | restartpoints_timed | restartpoints_req | restartpoints_done | write_time | sync_time | buffers_written | slru_written |          stats_reset
-----------+---------------+----------+---------------------+-------------------+--------------------+------------+-----------+-----------------+--------------+-------------------------------
        57 |             0 |       22 |                   0 |                 0 |                  0 |     588771 |      5422 |          210184 |           89 | 2026-10-28 08:19:41.420146+03
(1 row)
postgres=# SELECT * FROM pg_stat_wal;
 wal_records | wal_fpi | wal_bytes  | wal_buffers_full |          stats_reset
-------------+---------+------------+------------------+-------------------------------
     3855445 |  310173 | 2845947843 |            11322 | 2026-10-28 08:19:41.420153+03
(1 row)
postgres=# SELECT backend_type, object, context, writes, write_bytes, write_time
FROM pg_stat_io
WHERE object = 'wal';
    backend_type     | object | context | writes | write_bytes | write_time
---------------------+--------+---------+--------+-------------+------------
 client backend      | wal    | init    |      0 |           0 |          0
 client backend      | wal    | normal  | 219795 |  4542488576 |          0
 autovacuum launcher | wal    | init    |      0 |           0 |          0
 autovacuum launcher | wal    | normal  |      0 |           0 |          0
 autovacuum worker   | wal    | init    |      0 |           0 |          0
 autovacuum worker   | wal    | normal  |     19 |     1998848 |          0
 background worker   | wal    | init    |      0 |           0 |          0
 background worker   | wal    | normal  |      0 |           0 |          0
 walsender           | wal    | init    |      0 |           0 |          0
 walsender           | wal    | normal  |      0 |           0 |          0
 slotsync worker     | wal    | init    |      0 |           0 |          0
 slotsync worker     | wal    | normal  |      0 |           0 |          0
 standalone backend  | wal    | init    |      0 |           0 |          0
 standalone backend  | wal    | normal  |      0 |           0 |          0
 background writer   | wal    | init    |      0 |           0 |          0
 background writer   | wal    | normal  |      0 |           0 |          0
 checkpointer        | wal    | init    |      0 |           0 |          0
 checkpointer        | wal    | normal  |     22 |      286720 |          0
 io worker           | wal    | init    |      0 |           0 |          0
 io worker           | wal    | normal  |      0 |           0 |          0
 startup             | wal    | init    |      0 |           0 |          0
 startup             | wal    | normal  |      0 |           0 |          0
 walreceiver         | wal    | init    |      0 |           0 |          0
 walreceiver         | wal    | normal  |      0 |           0 |          0
 walsummarizer       | wal    | init    |      0 |           0 |          0
 walsummarizer       | wal    | normal  |      0 |           0 |          0
 walwriter           | wal    | init    |      0 |           0 |          0
 walwriter           | wal    | normal  |    341 |    20422656 |          0
(28 rows)
```
Рассчитайте объём WAL за 10 минут и средний объём WAL на одну контрольную точку;

```sh

WAL за 10 минут = wal_bytes(после) − wal_bytes(до)
2845947843 − 7959 = 2845939884 байт. 2845 МБ
Средний WAL на контрольную точку WAL за 10 минут / на контрольные точки
Средний WAL на контрольную точку = 2845939884 / 22 = 129360904 байт = 129 МБ

```
Проверьте по статистике/логам, выполнялись ли контрольные точки строго по расписанию, и объясните отклонения;
```sh
cat /var/log/postgresql/postgresql-18-pg_18.log | grep -a 'checkpoint starting: time'

2026-10-28 08:34:26.430 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:34:56.556 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:35:26.230 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:35:56.264 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:36:26.177 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:36:56.836 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:37:26.217 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:37:56.150 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:38:26.128 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:38:56.208 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:39:26.217 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:39:56.118 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:40:26.142 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:40:56.174 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:41:26.142 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:41:56.172 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:42:26.157 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:42:56.239 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:43:26.244 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:43:56.215 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:44:26.172 MSK [1479] LOG:  checkpoint starting: time
2026-10-28 08:45:56.327 MSK [1479] LOG:  checkpoint starting: time

# каждые 30 секунд запускаются
```

Сравните TPS при синхронном и асинхронном подтверждении коммита (2 прогона pgbench в сопоставимых условиях) и объясните разницу;
```sh

postgres@gorskiy-nn-VirtualBox:/var/log/postgresql$ pgbench -i -s 10 -U postgres otushw6
dropping old tables...
creating tables...
generating data (client-side)...
vacuuming...
creating primary keys...
done in 1.79 s (drop tables 0.04 s, create tables 0.01 s, client-side generate 1.33 s, vacuum 0.09 s, primary keys 0.32 s).
postgres@gorskiy-nn-VirtualBox:/var/log/postgresql$ PGOPTIONS='-c synchronous_commit=off' pgbench -c 8 -j 4 -T 60 -P 30 -U postgres otushw6
pgbench (18.6 (Ubuntu 18.6-1.pgdg26.04+2))
starting vacuum...end.
progress: 30.0 s, 10638.3 tps, lat 0.751 ms stddev 2.037, 0 failed
progress: 60.0 s, 9872.4 tps, lat 0.810 ms stddev 5.296, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 615337
number of failed transactions: 0 (0.000%)
latency average = 0.780 ms
latency stddev = 3.957 ms
initial connection time = 12.989 ms
tps = 10255.017090 (without initial connection time)
#статистика работы процесса checkpointer
SELECT * FROM pg_stat_checkpointer;
 num_timed | num_requested | num_done | restartpoints_timed | restartpoints_req | restartpoints_done | write_time | sync_time | buffers_written | slru_written |          stats_reset
-----------+---------------+----------+---------------------+-------------------+--------------------+------------+-----------+-----------------+--------------+-------------------------------
       278 |             0 |       25 |                   0 |                 0 |                  0 |     663820 |      9416 |          235090 |          134 | 2026-10-28 08:19:41.420146+03
postgres=# SELECT * FROM pg_stat_wal;
 wal_records | wal_fpi | wal_bytes  | wal_buffers_full |          stats_reset
-------------+---------+------------+------------------+-------------------------------
     7782813 |  352655 | 3547833410 |            21625 | 2026-10-28 08:19:41.420153+03
(1 row)
SELECT backend_type, object, context, writes, write_bytes, write_time
FROM pg_stat_io
WHERE object = 'wal';
    backend_type     | object | context | writes | write_bytes | write_time
---------------------+--------+---------+--------+-------------+------------
 client backend      | wal    | init    |      0 |           0 |          0
 client backend      | wal    | normal  | 230127 |  4660895744 |          0
 autovacuum launcher | wal    | init    |      0 |           0 |          0
 autovacuum launcher | wal    | normal  |      0 |           0 |          0
 autovacuum worker   | wal    | init    |      0 |           0 |          0
 autovacuum worker   | wal    | normal  |     48 |    18391040 |          0
 background worker   | wal    | init    |      0 |           0 |          0
 background worker   | wal    | normal  |      0 |           0 |          0
 walsender           | wal    | init    |      0 |           0 |          0
 walsender           | wal    | normal  |      0 |           0 |          0
 slotsync worker     | wal    | init    |      0 |           0 |          0
 slotsync worker     | wal    | normal  |      0 |           0 |          0
 standalone backend  | wal    | init    |      0 |           0 |          0
 standalone backend  | wal    | normal  |      0 |           0 |          0
 background writer   | wal    | init    |      0 |           0 |          0
 background writer   | wal    | normal  |      1 |     1941504 |          0
 checkpointer        | wal    | init    |      5 |    83886080 |          0
 checkpointer        | wal    | normal  |    213 |   115941376 |          0
 io worker           | wal    | init    |      0 |           0 |          0
 io worker           | wal    | normal  |      0 |           0 |          0
 startup             | wal    | init    |      0 |           0 |          0
 startup             | wal    | normal  |      0 |           0 |          0
 walreceiver         | wal    | init    |      0 |           0 |          0
 walreceiver         | wal    | normal  |      0 |           0 |          0
 walsummarizer       | wal    | init    |      0 |           0 |          0
 walsummarizer       | wal    | normal  |      0 |           0 |          0
 walwriter           | wal    | init    |     16 |   268435456 |          0
 walwriter           | wal    | normal  |   1016 |   483311616 |          0
(28 rows)
```

Сравните TPS при синхронном и асинхронном подтверждении коммита (2 прогона pgbench в сопоставимых условиях) и объясните разницу;

```sh
# асинхронный режим
postgres@gorskiy-nn-VirtualBox:/var/log/postgresql$ pgbench -i -s 10 -U postgres otushw6
dropping old tables...
creating tables...
generating data (client-side)...
vacuuming...
creating primary keys...
done in 1.79 s (drop tables 0.04 s, create tables 0.01 s, client-side generate 1.33 s, vacuum 0.09 s, primary keys 0.32 s).
postgres@gorskiy-nn-VirtualBox:/var/log/postgresql$ PGOPTIONS='-c synchronous_commit=off' pgbench -c 8 -j 4 -T 60 -P 30 -U postgres otushw6
pgbench (18.6 (Ubuntu 18.6-1.pgdg26.04+2))
starting vacuum...end.
progress: 30.0 s, 10638.3 tps, lat 0.751 ms stddev 2.037, 0 failed
progress: 60.0 s, 9872.4 tps, lat 0.810 ms stddev 5.296, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 615337
number of failed transactions: 0 (0.000%)
latency average = 0.780 ms
latency stddev = 3.957 ms
initial connection time = 12.989 ms
tps = 10255.017090 (without initial connection time)

#синхронный режим
postgres@gorskiy-nn-VirtualBox:/var/log/postgresql$ pgbench -i -s 10 -U postgres otushw6
dropping old tables...
creating tables...
generating data (client-side)...
vacuuming...
creating primary keys...
done in 2.11 s (drop tables 0.03 s, create tables 0.01 s, client-side generate 1.74 s, vacuum 0.06 s, primary keys 0.29 s).
postgres@gorskiy-nn-VirtualBox:/var/log/postgresql$ pgbench -c 8 -j 4 -T 60 -P 30 -U postgres otushw6
pgbench (18.6 (Ubuntu 18.6-1.pgdg26.04+2))
starting vacuum...end.
progress: 30.0 s, 923.6 tps, lat 8.652 ms stddev 11.952, 0 failed
progress: 60.0 s, 911.8 tps, lat 8.776 ms stddev 9.196, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 10
query mode: simple
number of clients: 8
number of threads: 4
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 55068
number of failed transactions: 0 (0.000%)
latency average = 8.714 ms
latency stddev = 10.672 ms
initial connection time = 15.521 ms
tps = 917.846581 (without initial connection time)


синхронный 917 tps,	асинхронный 10255 tps. synchronous_commit = on - 
при синхронном режиме commit дожидается записи данных WAL на диск( вызов fsync)
```
![]()

