Механизм блокировок

Цель:
диагностировать блокировки и взаимоблокировки по журналу сообщений и pg_locks;


Описание/Пошаговая инструкция выполнения домашнего задания:
Контекст


Дежурный DBA в «Ночной Ламе», во время акции в 20:00 получает лавину тикетов: админка «висит» на смене статуса заказа, курьеры приезжают раньше времени, а часть заказов остаётся в промежуточном состоянии. В мониторинге видно, что проблема не в CPU, а в том, что один процесс держит изменения открытыми слишком долго — и десятки других операций записи выстраиваются в очередь ожидания. Фишка сервиса — «антиспойлер-режим»: клиентский статус обновляется отдельно от курьерского, плюс в рамках одной операции одновременно правятся запись заказа и запись назначения курьера, поэтому разные потоки захватывают ресурсы в разном порядке. Руководитель просит за вечер найти источник очередей ожидания и объяснить, когда ситуация превращается в взаимоблокировку и что можно восстановить постфактум по журналу.

Настройте PostgreSQL так, чтобы в журнал сообщений записывались ожидания блокировок дольше 200 мс;
- выполнено
Воспроизведите ожидание блокировки дольше 200 мс и подтвердите это записью в журнале сообщений;
- выполнено
Откройте три сессии psql к одной базе данных;
- выполнено
Создайте тестовую таблицу и добавьте одну строку, которую будут обновлять все сессии;
- выполнено
Выполните в трёх сессиях команды UPDATE, обновляющие одну и ту же строку, так чтобы одна транзакция удерживала блокировку, а остальные ожидали;
- выполнено
Во время ожидания снимите список блокировок из представления pg_locks и сохраните его для отчёта;
- выполнено
Объясните смысл каждой блокировки из pg_locks (что блокируется, какой режим, кто держит, кто ожидает);
- выполнено
Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?
Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?

⭐️Задание со звёздочкой
Попробуйте воспроизвести ситуацию из пункта 9.


Примерное время выполнения 90–150 минут;

Настройте PostgreSQL так, чтобы в журнал сообщений записывались ожидания блокировок дольше 200 мс;
```sh
#Включаем логирование ожиданий блокировок
ALTER SYSTEM SET log_lock_waits = 'on';
#Устанавливаеv порог ожидания в 200ms
ALTER SYSTEM SET deadlock_timeout = '200ms';
       #Перечитываем конфиг
       SELECT pg_reload_conf();
```
Воспроизведите ожидание блокировки дольше 200 мс и подтвердите это записью в журнале сообщений;
```sh
CREATE TABLE hw8 (id  int PRIMARY KEY, val text);
INSERT INTO hw8 values (1, 'blocked');
    #Сессия 1
    BEGIN;
    UPDATE hw8 SET val = 'update_s1' where id = 1;
    #Сессия 2
    BEGIN;
    UPDATE hw8 SET val = 'update_s2' where id = 1;

2026-10-30 03:34:48.310 MSK [8157] postgres@otushw6 LOG:  process 8157 still waiting for ShareLock on transaction 1494311 after 200.787 ms
2026-10-30 03:34:48.310 MSK [8157] postgres@otushw6 DETAIL:  Process holding the lock: 8159. Wait queue: 8157.
2026-10-30 03:34:48.310 MSK [8157] postgres@otushw6 CONTEXT:  while updating tuple (0,2) in relation "hw8"
2026-10-30 03:34:48.310 MSK [8157] postgres@otushw6 STATEMENT:  UPDATE hw8 SET val = 'update_s2' where id = 1;
2026-10-30 03:34:50.429 MSK [1388] LOG:  checkpoint starting: time
2026-10-30 03:34:50.614 MSK [1388] LOG:  checkpoint complete: wrote 1 buffers (0.0%), wrote 0 SLRU buffers; 0 WAL file(s) added, 0 removed, 0 recycled; write=0.159 s, sync=0.003 s, total=0.186 s; sync files=1, longest=0.003 s, average=0.003 s; distance=0 kB, estimate=773 kB; lsn=2/14C3DB68, redo lsn=2/14C3DB08
```
Откройте три сессии psql к одной базе данных;
Создайте тестовую таблицу и добавьте одну строку, которую будут обновлять все сессии;

```sh
CREATE TABLE hww (id int PRIMARY KEY, val text);
INSERT INTO hww values (1, 'update');

```

Выполните в трёх сессиях команды UPDATE, обновляющие одну и ту же строку, так чтобы одна транзакция удерживала блокировку, а остальные ожидали;
Во время ожидания снимите список блокировок из представления pg_locks и сохраните его для отчёта;
Объясните смысл каждой блокировки из pg_locks (что блокируется, какой режим, кто держит, кто ожидает);
```sh
#session 1
otushw6=# BEGIN;
BEGIN
otushw6=*# update hw8 set val = 'update_s1hw8' where id =1;
UPDATE 1
otushw6=*# SELECT locktype, mode, granted, pid, pg_blocking_pids(pid) AS wait_for FROM pg_locks WHERE relation = 'hw8'::regclass;
 locktype |       mode       | granted | pid  | wait_for
----------+------------------+---------+------+----------
 relation | RowExclusiveLock | t       | 6122 | {}
(1 row)
# relation - блокировка на уровне таблицы
    # 	RowExclusiveLock - режим блокировки
        # t - блокировка true - процесс её держит
            # 6122 -  pid процесса 
                # wait_for - пусто никто не держит
#session 2
BEGIN;
update hw8 set val = 'up_s1hw8' where id =1;

#session 1
 SELECT locktype, mode, granted, pid, pg_blocking_pids(pid) AS wait_for FROM pg_locks WHERE relation = 'hw8'::regclass;
 locktype |       mode       | granted | pid  | wait_for
----------+------------------+---------+------+----------
 relation | RowExclusiveLock | t       | 6122 | {}
 relation | RowExclusiveLock | t       | 6136 | {6122}
 tuple    | ExclusiveLock    | t       | 6136 | {6122}
(3 rows)

# relation -  6122 блокировка на уровне таблицы первая сессия
# relation -  6136 блокировка на уровне таблицы вторая сессия,  wait_for ожидает выполнения 6122
# tuple блокировка строки - 6136 режим ExclusiveLock, wait_for ожидает выполнения 6122

#session 3
BEGIN;
update hw8 set val = 'down_s1hw8' where id =1;

#session 1
otushw6=*# SELECT locktype, mode, granted, pid, pg_blocking_pids(pid) AS wait_for FROM pg_locks WHERE relation = 'hw8'::regclass;
 locktype |       mode       | granted | pid  | wait_for
----------+------------------+---------+------+----------
 relation | RowExclusiveLock | t       | 6122 | {}
 relation | RowExclusiveLock | t       | 6136 | {6122}
 relation | RowExclusiveLock | t       | 6138 | {6136}
 tuple    | ExclusiveLock    | t       | 6136 | {6122}
 tuple    | ExclusiveLock    | f       | 6138 | {6136}
(5 rows)

# relation -  6122 блокировка на уровне таблицы первая сессия
# relation -  6136 блокировка на уровне таблицы вторая сессия,  wait_for ожидает выполнения 6122
# relation -  6138 блокировка на уровне таблицы третья сессия,  wait_for ожидает выполнения 6136
# tuple блокировка строки - 6136 режим ExclusiveLock, wait_for ожидает выполнения 6122
# tuple блокировка строки - 6138 режим ExclusiveLock, wait_for ожидает выполнения 6136


```
Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?
```sh
SELECT name, setting, unit, source, pending_restart
FROM pg_settings
WHERE name IN ('log_lock_waits', 'deadlock_timeout')
ORDER BY name;
       name       | setting | unit |       source       | pending_restart
------------------+---------+------+--------------------+-----------------
 deadlock_timeout | 200     | ms   | configuration file | f
 log_lock_waits   | on      |      | configuration file | f
(2 rows)

INSERT INTO hw8 (id, val) VALUES (4, 'a'), (5, 'b'), (6, 'c');
INSERT 0 3

#session 1
BEGIN;
UPDATE hw8 SET val='s1' WHERE id=4;
#session 2
BEGIN;
UPDATE hw8 SET val='s2' WHERE id=5;
#session 3
BEGIN;
UPDATE hw8 SET val='s3' WHERE id=6;
otushw6=*# SELECT locktype, mode, granted, pid, pg_blocking_pids(pid) AS wait_for FROM pg_locks WHERE relation = 'hw8'::regclass;
 locktype |       mode       | granted | pid  | wait_for
----------+------------------+---------+------+----------
 relation | RowExclusiveLock | t       | 6122 | {}
 relation | RowExclusiveLock | t       | 6136 | {}
 relation | RowExclusiveLock | t       | 6138 | {}
(3 rows)
# Не получил deadlock


# ПРОБУЕМ ЕЩЁ
#session 1
BEGIN;
UPDATE hw8 SET val='sR' WHERE id=4;
#session 2
BEGIN;
UPDATE hw8 SET val='sE' WHERE id=4;
#session 3
BEGIN;
UPDATE hw8 SET val='sh' WHERE id=4;

otushw6=*# SELECT locktype, mode, granted, pid, pg_blocking_pids(pid) AS wait_for FROM pg_locks WHERE relation = 'hw8'::regclass;
 locktype |       mode       | granted | pid  | wait_for
----------+------------------+---------+------+----------
 relation | RowExclusiveLock | t       | 6122 | {}
 relation | RowExclusiveLock | t       | 6136 | {6122}
 relation | RowExclusiveLock | t       | 6138 | {6136}
 tuple    | ExclusiveLock    | f       | 6138 | {6136}
 tuple    | ExclusiveLock    | t       | 6136 | {6122}
(5 rows)
# Не получил deadlock


# ПРОБУЕМ ЕЩЁ


#session 1
BEGIN;
UPDATE hw8 SET val='s1' WHERE id=1;
#session 2	
BEGIN;
UPDATE hw8 SET val='s2' WHERE id=2;	
#session 3
BEGIN;
UPDATE hw8 SET val='s3' WHERE id=3;	
#session 1
UPDATE hw8 SET val='s1b' WHERE id=2;	
#session 2	
UPDATE hw8 SET val='s2b' WHERE id=3;	
#session 3
UPDATE hw8 SET val='s3b' WHERE id=1;	

otushw6=*# UPDATE hw8 SET val='s3b' WHERE id=1;
ERROR:  deadlock detected
DETAIL:  Process 6138 waits for ShareLock on transaction 1494335; blocked by process 6122.
Process 6122 waits for ShareLock on transaction 1494336; blocked by process 6136.
Process 6136 waits for ShareLock on transaction 1494337; blocked by process 6138.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,10) in relation "hw8"

# Можно в журнале увидеть сообщения
ЛОГ:
2026-11-04 00:19:40.497 MSK [6136] postgres@otushw6 DETAIL:  Process holding the lock: 6138. Wait queue: 6136.
2026-11-04 00:19:40.497 MSK [6136] postgres@otushw6 CONTEXT:  while updating tuple (0,7) in relation "hw8"
2026-11-04 00:19:40.497 MSK [6136] postgres@otushw6 STATEMENT:  UPDATE hw8 SET val='s2b' WHERE id=3;
2026-11-04 00:19:54.625 MSK [6138] postgres@otushw6 LOG:  process 6138 detected deadlock while waiting for ShareLock on transaction 1494335 after 200.285 ms



```



Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?

```sh
#session 1
BEGIN;
UPDATE hw8 SET val = 'A';
#session 2
BEGIN;
UPDATE hw8 SET val = 'D';

otushw6=*# SELECT locktype, mode, granted, pid, pg_blocking_pids(pid) AS wait_for FROM pg_locks WHERE relation = 'hw8'::regclass;
 locktype |       mode       | granted | pid  | wait_for
----------+------------------+---------+------+----------
 relation | RowExclusiveLock | t       | 6122 | {}
 relation | RowExclusiveLock | t       | 6136 | {6122}
 tuple    | ExclusiveLock    | t       | 6136 | {6122}
(3 rows)
# Не блокируют, только создают очередь.
```
