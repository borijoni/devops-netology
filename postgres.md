# Домашнее задание к занятию "6.4. PostgreSQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.
```
Ответ:

Выполняем на заранее развернутом гостевом хосте c установленным Docker на virtualbox:
root@vagrant:/home/vagrant# docker run -d --rm --name my_postgres -e POSTGRES_PASSWORD=postgres -e PGDATA=/data/pgdata -it -p 5432:5432 -v /data/pgdata:/data/pgdata -v /data/backup:/data/backup postgres:13
cce76613bdebee9bdcf60d560b61c37a41959fc7d805acc7083047f32c665cc3
root@vagrant:/home/vagrant# docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                                       NAMES
cce76613bdeb   postgres:13   "docker-entrypoint.s…"   8 seconds ago   Up 7 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   my_postgres
```
Подключитесь к БД PostgreSQL используя `psql`.
```
Ответ:

root@vagrant:/home/vagrant# docker exec -it my_postgres psql -U postgres
psql (13.7 (Debian 13.7-1.pgdg110+1))
Type "help" for help.

postgres=#
```
Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:
- вывода списка БД
```
Ответ:

postgres-# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)
```
- подключения к БД
```
Ответ:

Подключение к БД (предварительно её создадим)
postgres=# CREATE DATABASE "test_database";
CREATE DATABASE
postgres=# \l
                                   List of databases
     Name      |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
---------------+----------+----------+------------+------------+-----------------------
 postgres      | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0     | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
               |          |          |            |            | postgres=CTc/postgres
 template1     | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
               |          |          |            |            | postgres=CTc/postgres
 test_database | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
(4 rows)

postgres=# \c test_database
You are now connected to database "test_database" as user "postgres".
test_database=#
```
- вывода списка таблиц
```
Ответ:

test_database=# \dt
Did not find any relations.
```
- вывода описания содержимого таблиц
```
Ответ:

test_database-# \dS+ pg_views
                         View "pg_catalog.pg_views"
   Column   | Type | Collation | Nullable | Default | Storage  | Description
------------+------+-----------+----------+---------+----------+-------------
 schemaname | name |           |          |         | plain    |
 viewname   | name |           |          |         | plain    |
 viewowner  | name |           |          |         | plain    |
 definition | text |           |          |         | extended |
View definition:
 SELECT n.nspname AS schemaname,
    c.relname AS viewname,
    pg_get_userbyid(c.relowner) AS viewowner,
    pg_get_viewdef(c.oid) AS definition
   FROM pg_class c
     LEFT JOIN pg_namespace n ON n.oid = c.relnamespace
  WHERE c.relkind = 'v'::"char";

```
- выхода из psql
```
Ответ:

test_database-# \q
root@vagrant:/home/vagrant#
```
## Задача 2

Используя `psql` создайте БД `test_database`.
```
Ответ:

Мы уже создали базу в прошлом задании командой:
postgres=# CREATE DATABASE "test_database";
CREATE DATABASE
```

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.
```
Ответ:

root@vagrant:/home/vagrant# docker exec -d my_postgres psql -U postgres -d test_database -f /data/backup/test_dump.sql
```

Перейдите в управляющую консоль `psql` внутри контейнера.
```
Ответ:

root@vagrant:/home/vagrant# docker exec -it my_postgres psql -U postgres
psql (13.7 (Debian 13.7-1.pgdg110+1))
Type "help" for help.

postgres=#

```
Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.
```
Ответ:

postgres=# \c test_database
You are now connected to database "test_database" as user "postgres".
test_database=# \dt
         List of relations
 Schema |  Name  | Type  |  Owner
--------+--------+-------+----------
 public | orders | table | postgres
(1 row)

test_database=# ANALYZE VERBOSE public.orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE
```
Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления и полученный результат.
```
Ответ:

test_database=# SELECT avg_width FROM pg_stats WHERE tablename='orders';
 avg_width
-----------
         4
        16
         4
(3 rows)
```
## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.
```
Ответ:

test_database=# ALTER TABLE orders RENAME TO orders_copy;
ALTER TABLE
test_database=# CREATE TABLE orders (LIKE orders_copy) PARTITION BY range(price);
CREATE TABLE
test_database=# CREATE TABLE orders_1 PARTITION OF orders FOR VALUES FROM (500) TO (MAXVALUE);
CREATE TABLE
test_database=# CREATE TABLE orders_2 PARTITION OF orders FOR VALUES FROM (MINVALUE) TO (500);
CREATE TABLE
test_database=# INSERT INTO orders SELECT * FROM orders_copy;
INSERT 0 8
test_database=# DROP TABLE orders_copy;
DROP TABLE
```
Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?
```
Ответ:

Исключить ручное разбиение таблицы можно было бы, если бы изначально её создали секционированной
```
## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.
```
Ответ:

docker exec -t my_postgres pg_dump -U postgres test_database -f /data/backup/bk_test_database.sql
```
Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?
```
Ответ:

Можно создать индекс
```
