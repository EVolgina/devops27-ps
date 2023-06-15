# Задача 1
- Используя Docker, поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.
- Подключитесь к БД PostgreSQL, используя psql.
- Воспользуйтесь командой \? для вывода подсказки по имеющимся в psql управляющим командам.
- Найдите и приведите управляющие команды для:
  -вывода списка БД,
  -подключения к БД,
  -вывода списка таблиц,
  -вывода описания содержимого таблиц,
  -выхода из psql.
### Ответ:
```
vagrant@server1:~/ps$ sudo docker run --rm --name pg -e POSTGRES_PASSWORD=admin -e POSTGRES_USER=admin -e POSTGRES_DB=pg -d -p 5433:5432 -v $HOME/docker/volumes/postgres:/var/lib/postgresql/data postgres
a59564101fc185c45e48d76766c483c02c9b21f5832870a40cb20795bf99fd91
vagrant@server1:~/ps$ sudo docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED          STATUS                          PORTS                                       NAMES
a59564101fc1   postgres                 "docker-entrypoint.s…"   56 seconds ago   Up 55 seconds                   0.0.0.0:5433->5432/tcp, :::5433->5432/tcp   pg
vagrant@server1:~/ps$ sudo docker exec -it a59564101fc1 psql -U admin -d pg
```
-вывода списка БД
```
pg=# \l
                                             List of databases
   Name    | Owner | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider | Access privileges
-----------+-------+----------+------------+------------+------------+-----------------+-------------------
 pg        | admin | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            |
 postgres  | admin | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            |
 template0 | admin | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/admin         +
           |       |          |            |            |            |                 | admin=CTc/admin
 template1 | admin | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            | =c/admin         +
           |       |          |            |            |            |                 | admin=CTc/admin
(4 rows)
```
-подключения к БД
```
pg=# \c postgres
You are now connected to database "postgres" as user "admin".
postgres=#
```
-вывода списка таблиц
```
postgres=# \dtS
                   List of relations
   Schema   |           Name           | Type  | Owner
------------+--------------------------+-------+-------
 pg_catalog | pg_aggregate             | table | admin
 pg_catalog | pg_am                    | table | admin
 pg_catalog | pg_amop                  | table | admin
 pg_catalog | pg_amproc                | table | admin
 pg_catalog | pg_attrdef               | table | admin
 pg_catalog | pg_attribute             | table | admin
 pg_catalog | pg_auth_members          | table | admin
 pg_catalog | pg_authid                | table | admin
 pg_catalog | pg_cast                  | table | admin
```
-вывода описания содержимого таблиц
```
postgres=# \dS+ pg_aggregate
                                          Table "pg_catalog.pg_aggregate"
      Column      |   Type   | Collation | Nullable | Default | Storage  | Compression | Stats target | Description
------------------+----------+-----------+----------+---------+----------+-------------+--------------+-------------
 aggfnoid         | regproc  |           | not null |         | plain    |             |              |
 aggkind          | "char"   |           | not null |         | plain    |             |              |
 aggnumdirectargs | smallint |           | not null |         | plain    |             |              |
 aggtransfn       | regproc  |           | not null |         | plain    |             |              |
 aggfinalfn       | regproc  |           | not null |         | plain    |             |              |
 aggcombinefn     | regproc  |           | not null |         | plain    |             |              |
 aggserialfn      | regproc  |           | not null |         | plain    |             |              |
 aggdeserialfn    | regproc  |           | not null |         | plain    |             |              |
 aggmtransfn      | regproc  |           | not null |         | plain    |             |              |
 aggminvtransfn   | regproc  |           | not null |         | plain    |             |              |
 aggmfinalfn      | regproc  |           | not null |         | plain    |             |              |
 aggfinalextra    | boolean  |           | not null |         | plain    |             |              |
 aggmfinalextra   | boolean  |           | not null |         | plain    |             |              |
 aggfinalmodify   | "char"   |           | not null |         | plain    |             |              |
 aggmfinalmodify  | "char"   |           | not null |         | plain    |             |              |
 aggsortop        | oid      |           | not null |         | plain    |             |              |
 aggtranstype     | oid      |           | not null |         | plain    |             |              |
 aggtransspace    | integer  |           | not null |         | plain    |             |              |
 aggmtranstype    | oid      |           | not null |         | plain    |             |              |
 aggmtransspace   | integer  |           | not null |         | plain    |             |              |
 agginitval       | text     | C         |          |         | extended |             |              |
 aggminitval      | text     | C         |          |         | extended |             |              |
Indexes:
    "pg_aggregate_fnoid_index" PRIMARY KEY, btree (aggfnoid)
Access method: heap
```
 -выхода из psql
```
postgres=# \q
vagrant@server1:~/ps$
```
# Задача 2
- Используя psql, создайте БД test_database.
- Изучите бэкап БД.
- Восстановите бэкап БД в test_database.
- Перейдите в управляющую консоль psql внутри контейнера.
- Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.
- Используя таблицу pg_stats, найдите столбец таблицы orders с наибольшим средним значением размера элементов в байтах.
- Приведите в ответе команду, которую вы использовали для вычисления, и полученный результат.
### Ответ:
```
pg=# CREATE DATABASE test_database; - создали базу
CREATE DATABASE
pg=# GRANT ALL PRIVILEGES ON DATABASE "test_database" to admin; создали пользователя с правами
GRANT
vagrant@server1:~/ps$ wget https://raw.githubusercontent.com/netology-code/virt-homeworks/virt-11/06-db-04-postgresql/test_data/test_dump.sql  - выкачали базу на хост
vagrant@server1:~/ps$ sudo docker cp /home/vagrant/ps/test_dump.sql 51045e62f695:/home/test_dump.sql  - скопировали
Successfully copied 180kB to 51045e62f695:/home/test_dump.sql
vagrant@server1:~/ps$ sudo docker exec -it 51045e62f695 bash
root@51045e62f695:/# psql -U admin -d test_database -f /home/test_dump.sql  - восттановили бэкап
SET
SET
SET
SET
SET
 set_config
------------
(1 row)
SET
SET
SET
SET
SET
SET
CREATE TABLE
psql:/home/test_dump.sql:34: ERROR:  role "postgres" does not exist
CREATE SEQUENCE
psql:/home/test_dump.sql:49: ERROR:  role "postgres" does not exist
ALTER SEQUENCE
ALTER TABLE
COPY 8
 setval
--------
      8
(1 row)
ALTER TABLE
``
- Подключаемся к базе, смотрим таблицы и запускаем ANALYZE
```
vagrant@server1:~/ps$ sudo docker exec -it 51045e62f695 psql -U admin -d pg
psql (15.3 (Debian 15.3-1.pgdg110+1))
Type "help" for help.

pg=# \c test_database
You are now connected to database "test_database" as user "admin".
test_database=# \dt
        List of relations
 Schema |  Name  | Type  | Owner
--------+--------+-------+-------
 public | orders | table | admin
(1 row)

test_database=# ANALYZE verbose orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE
```
- Найдем столбец таблицы orders с наибольшим средним значением размера элементов в байтах (это title)
```
test_database=# select attname, avg_width from pg_stats where tablename='orders';
 attname | avg_width
---------+-----------
 id      |         4
 title   |        16
 price   |         4
(3 rows)
```
# Задача 3
- Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и поиск по ней занимает долгое время. Вам как успешному выпускнику курсов DevOps в Нетологии предложили провести разбиение таблицы на 2: шардировать на orders_1 - price>499 и orders_2 - price<=499.
- Предложите SQL-транзакцию для проведения этой операции.\
Можно ли было изначально исключить ручное разбиение при проектировании таблицы orders?
### Ответ:

# Задача 4
Используя утилиту pg_dump, создайте бекап БД test_database.
Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца title для таблиц test_database?
### Ответ:
