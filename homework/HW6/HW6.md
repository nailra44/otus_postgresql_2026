Vacuum и autovacuum

Цель:
диагностировать увеличение размера таблиц и управлять его состоянием с помощью vacuum и autovacuum;


Описание/Пошаговая инструкция выполнения домашнего задания:

Контекст

В эксплуатации наблюдается ускоренный рост занимаемого места и постепенное замедление операций записи. По логам и метрикам нет явных «тяжёлых» запросов, но увеличиваются размеры таблиц и индексов без пропорционального роста полезных данных. Требуется проверить, как система освобождает место после массовых изменений строк, убедиться, что фоновые процессы обслуживания работают, и уметь воспроизвести ситуацию деградации при отключении автоматического обслуживания на конкретной таблице.


Описание / пошаговая инструкция выполнения ДЗ

Создайте ВМ (2 vCPU, 4 GB RAM, SSD 10 GB) и установите PostgreSQL 17 с дефолтными настройками;
- выполнено
Подготовьте тестовую базу pgbench (pgbench -i postgres) и выполните прогон нагрузки (pgbench -c8 -P 6 -T 60 -U postgres postgres);
- выполнено
Зафиксируйте состояние обслуживания после нагрузки: для пользовательских таблиц сохраните n_dead_tup, last_autovacuum, last_vacuum из pg_stat_user_tables;
- выполнено
Создайте таблицу с текстовым полем и заполните её 1 000 000 строк (допускается генерация через generate_series);
- выполнено
Зафиксируйте размер таблицы (например, pg_total_relation_size) и значение n_dead_tup;
- выполнено
Выполните 5 полных обновлений строк (добавление символа к текстовому полю), затем зафиксируйте n_dead_tup, last_autovacuum и размер таблицы;
- выполнено
Дождитесь срабатывания autovacuum (периодически проверяя last_autovacuum) и зафиксируйте изменения n_dead_tup/размера;
- выполнено срабатывает быстро изменения зафиксироаны в пункте выше
Отключите autovacuum только для этой таблицы, выполните 10 полных обновлений строк и снова зафиксируйте n_dead_tup и размер;
- выполнено
Объясните наблюдения: почему растёт размер и/или число «мёртвых» строк при отключённом обслуживании;
- выполнено
Включите autovacuum обратно для таблицы;
- выполнено


⭐️ Задание со звёздочкой

написать анонимный блок, который выполняет 10 итераций обновления таблицы в цикле и выводит номер итерации в сообщении.
- выполнено


Создайте ВМ (2 vCPU, 4 GB RAM, SSD 25 GB) и установите PostgreSQL 18 с дефолтными настройками;
```sql
#собираем с нуля
df -h
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           777M  4.8M  772M   1% /run
/dev/sda2        25G   12G   13G  48% /
tmpfs           1.9G  1.1M  1.9G   1% /dev/shm
none            1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
tmpfs           1.9G  8.0K  1.9G   1% /tmp
none            1.0M     0  1.0M   0% /run/credentials/systemd-resolved.service
tmpfs           389M   60K  389M   1% /run/user/60578
tmpfs           389M   52K  389M   1% /run/user/1000
    root@gorskiy-nn-VirtualBox:/home/gorskiy-nn# pg_lsclusters
    Ver Cluster Port Status Owner    Data directory               Log file
    18  pg_18   5858 online postgres /var/lib/postgresql/18/pg_18 /var/log/postgresql/postgresql-18-pg_18.log


```

Подготовьте тестовую базу pgbench (pgbench -i postgres) и выполните прогон нагрузки (pgbench -c8 -P 6 -T 60 -U postgres postgres);
```sh
CREATE DATABASE otushw6;
CREATE DATABASE

	   #Переключаемся на пользователя postgres
	       sudo su
                 root@gorskiy-nn-VirtualBox:/home/gorskiy-nn# su postgres
				    #инициализация pgbench
                        pgbench -i otushw6
						   #результат
						       dropping old tables...
                               NOTICE:  table "pgbench_accounts" does not exist, skipping
                               NOTICE:  table "pgbench_branches" does not exist, skipping
                               NOTICE:  table "pgbench_history" does not exist, skipping
                               NOTICE:  table "pgbench_tellers" does not exist, skipping
                               creating tables...
                               generating data (client-side)...
                               vacuuming...
                               creating primary keys...
                               done in 0.21 s (drop tables 0.00 s, create tables 0.04 s, client-side generate 0.09 s, vacuum 0.05 s, primary keys 0.04 s).
                                   #размер базы данных
                                       SELECT pg_size_pretty(pg_database_size('otushw6')) AS db_size;
                                             db_size
                                            ---------
                                             23 MB
                                                      
                                             #запуск нагрузки
                                             pgbench -c8 -P 6 -T 60 -U postgres otushw6
                                                 dropping old tables...
                                                  NOTICE:  table "pgbench_accounts" does not exist, skipping
                                                  NOTICE:  table "pgbench_branches" does not exist, skipping
                                                  NOTICE:  table "pgbench_history" does not exist, skipping
                                                  NOTICE:  table "pgbench_tellers" does not exist, skipping
                                                  creating tables...
                                                  generating data (client-side)...
                                                  vacuuming...                                                   
                                                  creating primary keys...
                                                  done in 0.21 s (drop tables 0.00 s, create tables 0.04 s, client-side generate 0.09 s, vacuum 0.05 s, primary keys 0.04 s).
                                                  postgres@gorskiy-nn-VirtualBox:/home/gorskiy-nn$ pgbench -c8 -P 6 -T 60 -U postgres otushw6
                                                  pgbench (18.6 (Ubuntu 18.6-1.pgdg26.04+2))
                                                  starting vacuum...end.
                                                  progress: 6.0 s, 483.6 tps, lat 16.460 ms stddev 13.684, 0 failed
                                                  progress: 12.0 s, 549.7 tps, lat 14.538 ms stddev 11.932, 0 failed
                                                  progress: 18.0 s, 463.7 tps, lat 17.267 ms stddev 12.069, 0 failed
                                                  progress: 24.0 s, 363.7 tps, lat 21.991 ms stddev 21.070, 0 failed
                                                  progress: 30.0 s, 472.1 tps, lat 16.941 ms stddev 12.445, 0 failed
                                                  progress: 36.0 s, 409.8 tps, lat 19.502 ms stddev 13.723, 0 failed
                                                  progress: 42.0 s, 506.7 tps, lat 15.783 ms stddev 10.716, 0 failed
                                                  progress: 48.0 s, 562.4 tps, lat 14.251 ms stddev 17.506, 0 failed
                                                  progress: 54.0 s, 531.6 tps, lat 15.012 ms stddev 10.375, 0 failed
                                                  progress: 60.0 s, 401.3 tps, lat 19.941 ms stddev 13.996, 0 failed
                                                  transaction type: <builtin: TPC-B (sort of)>
                                                  scaling factor: 1
                                                  query mode: simple
                                                  number of clients: 8
                                                  number of threads: 1
                                                  maximum number of tries: 1
                                                  duration: 60 s
                                                  number of transactions actually processed: 28476
                                                  number of failed transactions: 0 (0.000%)
                                                  latency average = 16.852 ms
                                                  latency stddev = 14.079 ms
                                                  initial connection time = 13.245 ms
                                                  tps = 474.512777 (without initial connection time)


```
Зафиксируйте состояние обслуживания после нагрузки: для пользовательских таблиц сохраните n_dead_tup, last_autovacuum, last_vacuum из pg_stat_user_tables;
```sh
 SELECT
    schemaname,
    relname AS table_name,
    n_dead_tup,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;
 schemaname |    table_name    | n_dead_tup |          last_vacuum          |        last_autovacuum
------------+------------------+------------+-------------------------------+-------------------------------
 public     | pgbench_accounts |       3702 | 2026-10-24 23:43:56.569314+03 | 2026-10-24 23:44:01.783163+03
 public     | pgbench_branches |          0 | 2026-10-24 23:50:44.216034+03 | 2026-10-24 23:52:02.162954+03
 public     | pgbench_tellers  |          0 | 2026-10-24 23:50:44.216125+03 | 2026-10-24 23:52:02.167963+03
 public     | pgbench_history  |          0 | 2026-10-24 23:43:56.588751+03 | 2026-10-24 23:52:02.174465+03


```
Создайте таблицу с текстовым полем и заполните её 1 000 000 строк (допускается генерация через generate_series);
```sh
CREATE TABLE test(string text);
    INSERT INTO test(string) SELECT 'notran' FROM generate_series(1,1000002);
    INSERT 0 1000002
          SELECT count(*) from test;
               count
               --------
               1000002
```
Зафиксируйте размер таблицы (например, pg_total_relation_size) и значение n_dead_tup;
```sh
                    SELECT pg_size_pretty(pg_total_relation_size('test'));
                         pg_size_pretty
                        ----------------
                         35 MB
                        (1 row)                        

                    SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum
                    FROM pg_stat_user_TABLEs WHERE relname = 'test';
                          relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
                         ---------+------------+------------+--------+-------------------------------
                          test    |    1000002 |          0 |      0 | 2026-10-25 00:06:02.405871+03
                         (1 row)
```
Выполните 5 полных обновлений строк (добавление символа к текстовому полю), затем зафиксируйте n_dead_tup, last_autovacuum и размер таблицы;

```sh
SELECT * FROM test ORDER BY string LIMIT 1 OFFSET 2;
SELECT * FROM test ORDER BY string LIMIT 1 OFFSET 3;
 string
--------
 notran

update test set string = 'tran' where string = 'notran';
UPDATE 1000002

update test set string = 'run' where string = 'tran';
update test set string = 'gun' where string = 'run';
update test set string = 'scan' where string = 'gun';
update test set string = 'plan' where string = 'scan';
     SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum
     FROM pg_stat_user_TABLEs WHERE relname = 'test';
      relname | n_live_tup | n_dead_tup | ratio% |       last_autovacuum
     ---------+------------+------------+--------+------------------------------
      test    |    1000002 |          0 |      0 | 2026-10-25 00:30:06.83383+03
     (1 row)
     SELECT pg_size_pretty(pg_total_relation_size('test'));
           pg_size_pretty
          ----------------
           173 MB
          (1 row)
          
    


```
Дождитесь срабатывания autovacuum (периодически проверяя last_autovacuum) и зафиксируйте изменения n_dead_tup/размера;
- выполнено срабатывает быстро изменения зафиксироаны в пункте выше

Отключите autovacuum только для этой таблицы, выполните 10 полных обновлений строк и снова зафиксируйте n_dead_tup и размер;
```sh

\dt
                List of tables
 Schema |       Name       | Type  |  Owner
--------+------------------+-------+----------
 public | pgbench_accounts | table | postgres
 public | pgbench_branches | table | postgres
 public | pgbench_history  | table | postgres
 public | pgbench_tellers  | table | postgres
 public | test             | table | postgres
(5 rows)

ALTER TABLE test SET (autovacuum_enabled = off);
    # проверка отключения autovacuum для таблицы
    SELECT relname, reloptions FROM pg_class WHERE relname = 'test';
        #результат
           relname |        reloptions
-          --------+--------------------------
           test    | {autovacuum_enabled=off}
          (1 row)
                # Текущее состояние                          
                SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum
                     FROM pg_stat_user_TABLEs WHERE relname = 'test';
                      relname | n_live_tup | n_dead_tup | ratio% | last_autovacuum
                ---------+------------+------------+--------+-----------------
                 test    |          0 |          0 |      0 |
                (1 row)
                    #10 обновлений
                        otushw6=# update test set string = '1' where string = 'plan';
                        UPDATE 1000002
                        otushw6=# update test set string = '2' where string = '1';
                        UPDATE 1000002
                        otushw6=# update test set string = '3' where string = '2';
                        UPDATE 1000002
                        otushw6=# update test set string = '4' where string = '3';
                        UPDATE 1000002
                        otushw6=# update test set string = '5' where string = '4';
                        UPDATE 1000002
                        otushw6=# update test set string = '6' where string = '5';
                        UPDATE 1000002
                        otushw6=# update test set string = '7' where string = '6';
                        UPDATE 1000002
                        otushw6=# update test set string = '8' where string = '7';
                        UPDATE 1000002
                        otushw6=# update test set string = '9' where string = '8';
                        UPDATE 1000002
                        otushw6=# update test set string = 'x' where string = '9';
                        UPDATE 1000002
                            # Состояние после обновлений 
                            SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum
                            FROM pg_stat_user_TABLEs WHERE relname = 'test';
                                 relname | n_live_tup | n_dead_tup |  ratio%   | last_autovacuum
                                ---------+------------+------------+-----------+-----------------
                                 test    |          0 |    9998706 | 999870600 |
                                (1 row)
                            # Живых строк не наблюдается в выводе
                                #Размер таблицы после создания 23 MB, после 5 update 173 MB, после 10 update
                                    otushw6=# SELECT pg_size_pretty(pg_total_relation_size('test'));
                                     pg_size_pretty
                                    ----------------
                                     487 MB
                                    (1 row)
# Просмотр порога срабатывания autovacuum
select c.relname,
current_setting('autovacuum_vacuum_threshold') as av_base_thresh,
current_setting('autovacuum_vacuum_scale_factor') as av_scale_factor,
(current_setting('autovacuum_vacuum_threshold')::int +
(current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples)) as av_thresh,
s.n_dead_tup
from pg_stat_user_tables s join pg_class c ON s.relname = c.relname
where s.n_dead_tup > (current_setting('autovacuum_vacuum_threshold')::int
+ (current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples));
    #Результат 9998706 накопленных мертвых строк
         relname | av_base_thresh | av_scale_factor |     av_thresh      | n_dead_tup
        ---------+----------------+-----------------+--------------------+------------
         test    | 50             | 0.2             | 200050.40000000002 |    9998706
        (1 row)
        # путь к таблице в data_directory
            otushw6=# SELECT pg_relation_filepath('test');
                 pg_relation_filepath
                 pg_relation_filepath
                ----------------------
                 base/24585/24612
                (1 row)
                # результат tuple нескольких первых строк. Пустые кортежи
                 SELECT lp as tuple, t_xmin, t_xmax, t_field3 as t_cid, t_ctid
                    FROM heap_page_items(get_raw_page('test',0));
                         tuple | t_xmin | t_xmax | t_cid | t_ctid
                        -------+--------+--------+-------+--------
                             1 |        |        |       |
                             2 |        |        |       |
                             3 |        |        |       |
                             4 |        |        |       |
                             5 |        |        |       |
                             6 |        |        |       |
                             7 |        |        |       |
                             8 |        |        |       |

```
Объясните наблюдения: почему растёт размер и/или число «мёртвых» строк при отключённом обслуживании;
Включите autovacuum обратно для таблицы;

```sh
# число «мёртвых» строк при отключённом autovacuum растет потому что не очищаются "мертвые строки" автоматически, выделяемое место под таблицу растёт и будет расти независимо от включения autovacuum. Место можно уменьшить только выполнив vacuum full - пересоздание таблицы, на момент выполнения  vacuum full таблица будет недоступна для работы. 
ALTER TABLE test SET (autovacuum_enabled = on);

    # После включения autovacuum отработал за минуту

    SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum
                            FROM pg_stat_user_TABLEs WHERE relname = 'test';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
---------+------------+------------+--------+-------------------------------
 test    |     978479 |          0 |      0 | 2026-10-25 22:25:38.084113+03
(1 row)

otushw6=# select c.relname,
current_setting('autovacuum_vacuum_threshold') as av_base_thresh,
current_setting('autovacuum_vacuum_scale_factor') as av_scale_factor,
(current_setting('autovacuum_vacuum_threshold')::int +
(current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples)) as av_thresh,
s.n_dead_tup
from pg_stat_user_tables s join pg_class c ON s.relname = c.relname
where s.n_dead_tup > (current_setting('autovacuum_vacuum_threshold')::int
+ (current_setting('autovacuum_vacuum_scale_factor')::float * c.reltuples));
 relname | av_base_thresh | av_scale_factor | av_thresh | n_dead_tup
---------+----------------+-----------------+-----------+------------
(0 rows)
    # размер таблицы не изменился
    SELECT pg_size_pretty(pg_total_relation_size('test'));
     pg_size_pretty
      ----------------
       487 MB
     (1 row)
        # Пылесосим
        vacuum full test;
            otushw6=# SELECT pg_size_pretty(pg_total_relation_size('test'));
             pg_size_pretty
            ----------------
             35 MB
            (1 row)
# результат tuple нескольких первых строк.  
 SELECT lp as tuple, t_xmin, t_xmax, t_field3 as t_cid, t_ctid
FROM heap_page_items(get_raw_page('test',0));
 tuple | t_xmin | t_xmax | t_cid | t_ctid
-------+--------+--------+-------+---------
     1 |  29289 |      0 |     0 | (0,1)
     2 |  29289 |      0 |     0 | (0,2)
     3 |  29289 |      0 |     0 | (0,3)
     4 |  29289 |      0 |     0 | (0,4)
     5 |  29289 |      0 |     0 | (0,5)
     6 |  29289 |      0 |     0 | (0,6)
     7 |  29289 |      0 |     0 | (0,7)
     8 |  29289 |      0 |     0 | (0,8)

```
написать анонимный блок, который выполняет 10 итераций обновления таблицы в цикле и выводит номер итерации в сообщении.
```sh
otushw6=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum
                            FROM pg_stat_user_TABLEs WHERE relname = 'test';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
---------+------------+------------+--------+-------------------------------
 test    |     978479 |          0 |      0 | 2026-10-25 22:25:38.084113+03
(1 row)

otushw6=# SELECT lp as tuple, t_xmin, t_xmax, t_field3 as t_cid, t_ctid
FROM heap_page_items(get_raw_page('test',0));
otushw6=# DO $$
BEGIN
    FOR i IN 1..10 LOOP
      UPDATE test set string = 'x';
      RAISE NOTICE 'interation №: %', i;
    END LOOP;

END $$;
NOTICE:  interation №: 1
NOTICE:  interation №: 2
NOTICE:  interation №: 3
NOTICE:  interation №: 4
NOTICE:  interation №: 5
NOTICE:  interation №: 6
NOTICE:  interation №: 7
NOTICE:  interation №: 8
NOTICE:  interation №: 9
NOTICE:  interation №: 10
DO
otushw6=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum
                            FROM pg_stat_user_TABLEs WHERE relname = 'test';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
---------+------------+------------+--------+-------------------------------
 test    |     978479 |   10000020 |   1021 | 2026-10-25 22:25:38.084113+03
(1 row)


otushw6=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum
                            FROM pg_stat_user_TABLEs WHERE relname = 'test';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
---------+------------+------------+--------+-------------------------------
 test    |     977954 |          0 |      0 | 2026-10-25 22:44:56.535167+03

 otushw6=# select count(*) from test;
  count
---------
 1000002
(1 row)


```



![]()

