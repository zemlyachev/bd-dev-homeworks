# Домашняя работа к занятию 4. «PostgreSQL»

## Задача 1

Используя Docker, поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL, используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:

- вывода списка БД,
```shell
 \l[+]   [PATTERN]      list databases
```  
- подключения к БД,
```shell
 \c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}
                       connect to new database (currently "postgres")
```
- вывода списка таблиц,
```shell
 \dt[S+] [PATTERN]      list tables
 # или 
 \d[S+]                 list tables, views, and sequences
```
- вывода описания содержимого таблиц,
```shell
 \d[S+]  NAME           describe table, view, sequence, or index
```
- выхода из psql.
```shell
 \q                     quit psql
```

## Задача 2

Используя `psql`, создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.
```shell
test_database=# analyze verbose public.orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 0 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE
```

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления, и полученный результат.

```sql
test_database=# SELECT attname, avg_width
test_database-# FROM pg_stats
test_database-# WHERE tablename = 'orders'
test_database-# ORDER BY avg_width DESC
test_database-# LIMIT 1;
 attname | avg_width 
---------+-----------
 title   |        16
(1 row)

```

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам как успешному выпускнику курсов DevOps в Нетологии предложили
провести разбиение таблицы на 2: шардировать на orders_1 - price>499 и orders_2 - price<=499.

Предложите SQL-транзакцию для проведения этой операции.

```sql
test_database=# START TRANSACTION;
START TRANSACTION
test_database=*# CREATE TABLE orders_1 (CHECK (price > 499)) INHERITS (orders);
CREATE TABLE
test_database=*# INSERT INTO orders_1 SELECT * FROM orders WHERE price > 499;
INSERT 0 3
test_database=*# CREATE TABLE orders_2 (CHECK (price <= 499)) INHERITS (orders);
CREATE TABLE
test_database=*# INSERT INTO orders_2 SELECT * FROM orders WHERE price <= 499;
INSERT 0 5
test_database=# DELETE FROM ONLY orders;
DELETE 8
test_database=*# COMMIT;
COMMIT
```

Можно ли было изначально исключить ручное разбиение при проектировании таблицы orders?

> Да, прописав правила
```sql
CREATE RULE orders_1 AS ON INSERT TO orders WHERE ( price > 499 ) DO INSTEAD INSERT INTO orders_1 VALUES (NEW.*);
CREATE RULE orders_2 AS ON INSERT TO orders WHERE ( price <= 499 ) DO INSTEAD INSERT INTO orders_2 VALUES (NEW.*);
```
## Задача 4

Используя утилиту `pg_dump`, создайте бекап БД `test_database`.

```shell
root@9b6ef62dc07f:/# pg_dump -U postgres test_database > /tmp/test_database_backup.sql
root@9b6ef62dc07f:/# ls /tmp
test_database_backup.sql  test_dump.sql
```

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

> В [скрипте](test_data/test_database_backup.sql) добавил бы `UNIQUE` для поля `title` при создании таблиц
```sql
title character varying(80) NOT NULL UNIQUE
```

