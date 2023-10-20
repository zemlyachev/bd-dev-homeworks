# Домашняя работа к занятию 3. «MySQL»

## Задача 1

Используя Docker, поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-03-mysql/test_data) и 
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h`, получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из её вывода версию сервера БД.

```sh
mysql> \s
--------------
mysql  Ver 8.0.34 for Linux on x86_64 (MySQL Community Server - GPL)

Connection id:          9
Current database:       
Current user:           root@localhost
SSL:                    Not in use
Current pager:          stdout
Using outfile:          ''
Using delimiter:        ;
Server version:         8.0.34 MySQL Community Server - GPL # <-- ВЕРСИЯ СЕРВЕРА БД
Protocol version:       10
Connection:             Localhost via UNIX socket
Server characterset:    utf8mb4
Db     characterset:    utf8mb4
Client characterset:    latin1
Conn.  characterset:    latin1
UNIX socket:            /var/run/mysqld/mysqld.sock
Binary data as:         Hexadecimal
Uptime:                 2 min 58 sec

Threads: 2  Questions: 35  Slow queries: 0  Opens: 138  Flush tables: 3  Open tables: 56  Queries per second avg: 0.196
--------------
```

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество(!) записей с `price` > 300.

```sh
mysql> \u test_db
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SELECT COUNT(*) AS records_count FROM orders WHERE price > 300;
+---------------+
| records_count |
+---------------+
|             1 |  # <-- Всего одна запись с price > 300
+---------------+
1 row in set (0.00 sec)
```

## Задача 2

Создайте пользователя `test` в БД c паролем `test-pass`, используя:

- плагин авторизации `mysql_native_password`
- срок истечения пароля — 180 дней 
- количество попыток авторизации — 3 
- максимальное количество запросов в час — 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James".

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
    
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES, получите данные по пользователю `test` и 
**приведите в ответе к задаче**.

```sql
# Создаем
mysql> CREATE USER 'test'@'localhost' 
    -> IDENTIFIED WITH mysql_native_password BY 'test-pass' 
    -> WITH MAX_QUERIES_PER_HOUR 100 
    -> PASSWORD EXPIRE INTERVAL 180 DAY 
    -> FAILED_LOGIN_ATTEMPTS 3 
    -> PASSWORD_LOCK_TIME 30 
    -> ATTRIBUTE '{"firstname": "James", "lastname": "Pretty"}';
Query OK, 0 rows affected (0.01 sec)

# Привилегии
mysql> GRANT SELECT ON test_db.* TO test@localhost;
Query OK, 0 rows affected, 1 warning (0.01 sec)

# Данные по пользователю
mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES
    -> WHERE USER='test' and HOST='localhost';
+------+-----------+----------------------------------------------+
| USER | HOST      | ATTRIBUTE                                    |
+------+-----------+----------------------------------------------+
| test | localhost | {"lastname": "Pretty", "firstname": "James"} |
+------+-----------+----------------------------------------------+
1 row in set (0.00 sec)
```


## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

```sh
mysql> SELECT TABLE_SCHEMA, TABLE_NAME, ENGINE 
    -> FROM INFORMATION_SCHEMA.TABLES 
    -> WHERE TABLE_SCHEMA = 'test_db';
+--------------+------------+--------+
| TABLE_SCHEMA | TABLE_NAME | ENGINE |
+--------------+------------+--------+
| test_db      | orders     | InnoDB |
+--------------+------------+--------+
1 row in set (0.00 sec)
```

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`,
- на `InnoDB`.

```sh
mysql> SHOW PROFILES;
+----------+------------+------------------------------------+
| Query_ID | Duration   | Query                              |
+----------+------------+------------------------------------+
|        1 | 0.03663125 | ALTER TABLE orders ENGINE = MyISAM |
|        2 | 0.03720650 | ALTER TABLE orders ENGINE = InnoDB |
|        3 | 0.03487000 | ALTER TABLE orders ENGINE = MyISAM |
|        4 | 0.03751500 | ALTER TABLE orders ENGINE = InnoDB |
+----------+------------+------------------------------------+
4 rows in set, 1 warning (0.00 sec)
```


## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):

- скорость IO важнее сохранности данных;
- нужна компрессия таблиц для экономии места на диске;
- размер буффера с незакомиченными транзакциями 1 Мб;
- буффер кеширования 30% от ОЗУ;
- размер файла логов операций 100 Мб.

Приведите в ответе изменённый файл `my.cnf`.

```ini
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL

# Custom config should go here
!includedir /etc/mysql/conf.d/

innodb_flush_log_at_trx_commit = 2
innodb_file_per_table = 1
innodb_log_buffer_size = 1M
innodb_buffer_pool_size = 666M
innodb_log_file_size = 100M
```




