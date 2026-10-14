Домашнее задание

Цель:
настроить роль только для чтения в PostgreSQL и устранить типовые ошибки доступа, связанные со схемами и правами по умолчанию;


Описание/Пошаговая инструкция выполнения домашнего задания:
Контекст

В компании выделяется роль для аудиторов и аналитиков: требуется доступ только на чтение к данным в отдельной схеме, без возможности создавать объекты и модифицировать данные; при этом команда регулярно сталкивается с инцидентами «права выданы, но таблица не читается» из-за того, что объекты создаются не в той схеме или запросы выполняются без явного указания схемы;


Описание / пошаговая инструкция выполнения ДЗ:

Создайте кластер PostgreSQL 17 и базу testdb;
- выполнено
Создайте схему testnm, таблицу t1(c1 int) и вставьте строку;
- выполнено
Создайте роль readonly, выдайте ей CONNECT к testdb, USAGE на testnm, SELECT на таблицы схемы testnm;
- выполнено
Создайте пользователя testread/test123, назначьте роль readonly, выполните select * from t1; и зафиксируйте результат;
- test123 - пароль?
Пересоздайте таблицу как testnm.t1, проверьте select * from testnm.t1; и настройте поведение, чтобы обращение к t1 было предсказуемым;
- ммм - я создал сначала в схеме public, потом в схеме testnm подойдёт?
Под testread проверьте попытку create table и insert; при необходимости запретите и подтвердите запрет повторной проверкой.
- выполнено


Создайте кластер PostgreSQL 17 и базу testdb;
```sh
# Для подключения по ssh
sudo apt install openssl-server
# Для удобства
sudo apt install vim
# Установка postgresql
sudo apt install -y postgresql-common
    # подключение репозитория
    sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh  
        sudo apt install -y postgresql-18
            sudo pg_dropcluster --stop 18 main
                sudo pg_createcluster 18 pg_18 --port 5458
                    sudo pg_ctlcluster 18 pg_18 start
                        sudo -u postgres psql --port 5458
                            \password
                                SHOW hba_file;
                                    SHOW config_file;   
                                        vi /etc/postgresql/18/pg_18/pg_hba.conf
                                           # результат
                                           # IPv4 local connections:
                                           host    all             all             all                     scram-sha-256
                                               vi /etc/postgresql/18/pg_18/postgresql.conf
                                                   #результат
                                                   listen_addresses = '*'          # what IP address(es) to listen on;
                                                       sudo pg_ctlcluster 18 pg_18 restart
                                                           CREATE DATABASE testdb;
                                 
```
Создайте схему testnm, таблицу t1(c1 int) и вставьте строку;
```sh
# вывод списка со схемами
\dn
    #результат
          List of schemas
      Name  |       Owner
    --------+-------------------
     public | pg_database_owner
    (1 row)
        CREATE TABLE t1(c1 int);
        #результат
        CREATE TABLE
           INSERT INTO t1 VALUES (1);
           INSERT INTO t1 VALUES (3);
           INSERT INTO t1 VALUES (4);

```
Создайте роль readonly, выдайте ей CONNECT к testdb, USAGE на testnm, SELECT на таблицы схемы testnm;
```sh
# Создайте роль readonly
CREATE ROLE readonly LOGIN PASSWORD '111'
   #просмотр ролей
   \du
                                            List of roles
         Role name |                         Attributes
        -----------+------------------------------------------------------------
         postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS
         readonly  |

   # проверка подключения
   sudo -u readonly psql --port 5458
       sudo: user 'readonly' not found
# выдайте ей CONNECT к testdb
GRANT CONNECT ON DATABASE testdb TO readonly;
    # 
    \l+ testdb
    postgres=# \l+
    postgres=# \l+ testdb
                                                                                  List of databases
              Name  |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | Locale | ICU Rules |               Access privileges   |  Size   | Tablespace | Description
            --------+----------+----------+-----------------+-------------+-------------+--------+-----------            +-----------------------+---------+------------+-------------
             testdb | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =Tc/            postgres         +| 7617 kB | pg_default |
                    |          |          |                 |             |             |        |           |             postgres=CTc/postgres+|         |            |
                    |          |          |                 |             |             |        |           |             readonly=c/postgres   |         |            |
            (1 row)
                      # Справка
                Буква	Привелегии	Описание
                  C	CREATE	создавать объекты (таблицы, схемы) в этой базе
                  T	TEMPORARY	создавать временные таблицы в этой базе
                  c	CONNECT	подключаться к этой базе
                   Для таблиц (\dp, GRANT ... ON TABLE)
                  r	SELECT	читать строки (SELECT)
                  a	INSERT	добавлять строки (INSERT)
                  w	UPDATE	изменять строки (UPDATE)
                  d	DELETE	удалять строки (DELETE)
                  D	TRUNCATE	очищать таблицу (TRUNCATE)
                  x	REFERENCES	создавать внешние ключи на эту таблицу
                  t	TRIGGER	создавать триггеры на эту таблицу
                  Для схем (\dn+, GRANT ... ON SCHEMA)
                  U	USAGE	«видеть» объекты в схеме (без этого SELECT не сработает)
                  C	CREATE	создавать объекты в этой схеме
                  Для последовательностей (\dp, GRANT ... ON SEQUENCE)
                  r	SELECT	читать текущее значение (currval, nextval)
                  w	UPDATE	менять значение (setval)
                  U	USAGE	использовать (nextval, currval)
                   Для функций (\df+, GRANT ... ON FUNCTION)
                  X	EXECUTE	вызывать функцию   

   # проверка подключения
    psql -U readonly -h localhost -p 5458
       Password for user readonly:
            psql: error: connection to server at "localhost" (127.0.0.1), port 5458 failed: FATAL:  database "readonly" does not exist
    psql -U readonly -h localhost -p 5458 -d postgres
        Password for user readonly:
            psql (18.6 (Ubuntu 18.6-1.pgdg26.04+2))
            SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN:     postgresql)
            Type "help" for help.
            postgres=>
        #под кем подключение
        postgres=> \conninfo
                        Connection Information
                  Parameter       |         Value
            ----------------------+------------------------
             Database             | postgres
             Client User          | readonly
             Host                 | localhost
             Host Address         | 127.0.0.1
             Server Port          | 5458
             Options              |
             Protocol Version     | 3.0
             Password Used        | true
             GSSAPI Authenticated | false
             Backend PID          | 16295
             SSL Connection       | true
             SSL Library          | OpenSSL
             SSL Protocol         | TLSv1.3
             SSL Key Bits         | 256
             SSL Cipher           | TLS_AES_256_GCM_SHA384
             SSL Compression      | false
             ALPN                 | postgresql
             Superuser            | off
             Hot Standby          | off
            (19 rows)
            
        # или под кем подключение
        SELECT current_user, current_database(), inet_server_port();
            # пробуем хулиганить
            CREATE DATABASE ttt;
                ERROR:  permission denied to create database
            CREATE TABLE t2(c1 int);
                ERROR:  permission denied for schema public
                LINE 1: CREATE TABLE t2(c1 int);
            SELECT * FROM pg_catalog.pg_authid;
                ERROR:  permission denied for table pg_authid
            ALTER ROLE readonly SUPERUSER;
                ERROR:  permission denied to alter role
                DETAIL:  Only roles with the SUPERUSER attribute may change the SUPERUSER attribute.
            #подключение к тестовой бд
            \c testdb
                SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
                You are now connected to database "testdb" as user "readonly".
                testdb=>
                # 
                CREATE TABLE t2(c1 int);
                    ERROR:  permission denied for schema public
                    LINE 1: CREATE TABLE t2(c1 int);    
                        testdb=> \dt
                                  List of tables
                         Schema | Name | Type  |  Owner
                        --------+------+-------+----------
                         public | t1   | table | postgres
                        (1 row)
                            testdb=> select * from t1;
                                ERROR:  permission denied for table t1
# выдать USAGE на testnm, SELECT на таблицы схемы testnm;
    # Так забыл схему testnm создать, возвращаемся и дропаем роль
    DROP ROLE readonly;
       postgres=# DROP ROLE readonly;
            ERROR:  role "readonly" cannot be dropped because some objects depend on it
            DETAIL:  privileges for database testdb
                REVOKE CONNECT ON DATABASE testdb FROM readonly;
                   REVOKE
                        DROP ROLE readonly;
                            DROP ROLE
                                \с tetdb 
                                    DROP TABLE t1;
                                       DROP TABLE
#  Создаём схему подключившишь к базе
с\ testdb 
CREATE SCHEMA testnm;
    # ПРосмотр схемы
        \dn
               List of schemas
              Name  |       Owner
            --------+-------------------
             public | pg_database_owner
             testnm | postgres
            (2 rows)
# выполняем действия описанные выше 
    CREATE TABLE t1(c1 int);
        INSERT INTO t1 VALUES (1);
        INSERT INTO t1 VALUES (3);
        INSERT INTO t1 VALUES (4);
            CREATE ROLE readonly LOGIN PASSWORD '111';
               GRANT CONNECT ON DATABASE testdb TO readonly;
               #Так, таблица создалась в схеме public, роли нет  
                   testdb=# DROP TABLE t1;
                   DROP TABLE
#Создаём таблицу в схеме
CREATE TABLE testnm.t1(c1 int);
    CREATE TABLE
        INSERT INTO testnm.t1 VALUES (1);
        INSERT INTO testnm.t1 VALUES (3);
        INSERT INTO testnm.t1 VALUES (4);
           CREATE ROLE readonly LOGIN PASSWORD '111';
                CREATE ROLE
                   GRANT CONNECT ON DATABASE testdb TO readonly;
                        GRANT
#USAGE на testnm, SELECT на таблицы схемы testnm;
    GRANT USAGE ON SCHEMA testnm TO readonly;
        GRANT SELECT ON testnm.t1 TO readonly; #выдаём для конкретной таблицы
            SELECT * FROM testnm.t1;
                 c1
                ----
                  1
                  3
                  4
                (3 rows)
                    CREATE TABLE testnm.t2(c2 int);
                        ERROR:  permission denied for schema testnm
                        LINE 1: CREATE TABLE testnm.t2(c2 int);

```
Создайте пользователя testread/test123, назначьте роль readonly, выполните select * from t1; и зафиксируйте результат;
```sh
CREATE USER testread PASSWORD 'test123';
    ERROR:  permission denied to create role
    DETAIL:  Only roles with the CREATEROLE attribute may create roles.
        \c postgres postgres
            CREATE USER testread PASSWORD 'test123';
                CREATE ROLE
                    CREATE USER testread PASSWORD 'test123';
                        CREATE ROLE
                            GRANT readonly TO testread;
                               \du
                               \l+ testdb
                                                                          List of databases
                                   Name  |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | Locale | ICU Rules |                                    Access privileges   |  Size   | Tablespace | Description
                                 --------+----------+----------+-----------------+-------------+-------------+--------+-----------                                 +-----------------------+---------+------------+-------------
                                  testdb | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8 |        |           | =Tc/                                 postgres         +| 7774 kB | pg_default |
                                         |          |          |                 |             |             |        |           |                                  postgres=CTc/postgres+|         |            |
                                         |          |          |                 |             |             |        |           |                                  readonly=c/postgres   |         |            |
                                         (1 row)
#подключаемся
psql -U testread -h localhost --port 5458
    Password for user testread:
    psql: error: connection to server at "localhost" (127.0.0.1), port 5458 failed: FATAL:  database "testread" does not exist
        psql -U testread -h localhost -d testdb --port 5458
            Password for user testread:
            psql (18.6 (Ubuntu 18.6-1.pgdg26.04+2))
            SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
            Type "help" for help.
            testdb=>
                testdb=> SELECT user;
                       user
                    ----------
                     testread
                    (1 row)
                        testdb=> select * from testnm.t1;
                             c1
                            ----
                              1
                              3
                              4
                            (3 rows)

```
Под testread проверьте попытку create table и insert; при необходимости запретите и подтвердите запрет повторной проверкой.
```sh
CREATE TABLE testnm.t3(c3 int); 
    testdb=> CREATE TABLE testnm.t3(c3 int);
    ERROR:  permission denied for schema testnm
    LINE 1: CREATE TABLE testnm.t3(c3 int);
        testdb=>         INSERT INTO testnm.t1 VALUES (5);
        INSERT INTO testnm.t1 VALUES (8);
        INSERT INTO testnm.t1 VALUES (9);
            ERROR:  permission denied for table t1
            ERROR:  permission denied for table t1
            ERROR:  permission denied for table t1
```
Недопонял смысл "чтобы обращение к t1 было предсказуемым;" 
Пересоздайте таблицу как testnm.t1, проверьте select * from testnm.t1; и настройте поведение, чтобы обращение к t1 было предсказуемым;
```sh
#Надо создать роль так, чтобы при выполнении select * from t1; не было ошибки. Правильно? 

postgres=# ALTER ROLE readonly SET search_path TO testnm, public;
ALTER ROLE

psql -U testread -h localhost -d testdb --port 5458
    Password for user testread:
    psql (18.6 (Ubuntu 18.6-1.pgdg26.04+2))
    SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
    Type "help" for help.
    testdb=>
          testdb=> select user;
                 user
                 ----------
                  testread
                 (1 row)
                     select * from t1;
                     # Не работает, привелегии не наследуются от роли пользователю
                        # Для USER выдаем привелегии
                         postgres=# ALTER USER testread SET search_path TO testnm, public;
                                   ALTER ROLE
                            #Фуф заработало
                                   testdb=> select * from t1;
                                    c1
                                   ----
                                     1
                                     3
                                     4
                                   (3 rows)
   

```
![](End.jpg)