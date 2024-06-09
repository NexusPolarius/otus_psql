# Репликация

## Подготовка к выполнению ДЗ
Созданы три ВМ в VirtualBox с ОС Ubuntu 20.04. На машинах установлен PostgreSQL 15, отредактирован pg_hba.conf, postgresql.conf, wal_level установлен как logical.

## Выполнение ДЗ

0. Пошаговая инструкция из ДЗ на мой взгляд не очень корректна и логична(подписка на не существующую публикацию), поэтому немного изменил порядок выполняемых действий.

1. Создаем таблицы test и test2 на трех ВМ.
	
```
postgres=# CREATE TABLE test (id SERIAL PRIMARY KEY, title TEXT);
CREATE TABLE

postgres=# CREATE TABLE test2 (id SERIAL PRIMARY KEY, title TEXT);
CREATE TABLE
```

2. Создаем публикации.

На ВМ1:

```
postgres=# CREATE PUBLICATION vm1_test_pub FOR TABLE test;
CREATE PUBLICATION

postgres=# \dRp+
                          Publication vm1_test_pub
  Owner   | All tables | Inserts | Updates | Deletes | Truncates | Via root
----------+------------+---------+---------+---------+-----------+----------
 postgres | f          | t       | t       | t       | t         | f
Tables:
    "public.test"
```

На ВМ2:

```
postgres=# CREATE PUBLICATION vm2_test2_pub FOR TABLE test2;
CREATE PUBLICATION

postgres=# \dRp+
                         Publication vm2_test2_pub
  Owner   | All tables | Inserts | Updates | Deletes | Truncates | Via root
----------+------------+---------+---------+---------+-----------+----------
 postgres | f          | t       | t       | t       | t         | f
Tables:
    "public.test2"
```

3. Подписываемся на публикации.

На ВМ1 подписываемся на таблицу test2 ВМ2:

```
postgres=# CREATE SUBSCRIPTION vm1_test2_sub CONNECTION 'host=10.0.1.xxx user=postgres dbname=postgres' PUBLICATION vm2_test2_pub WITH (copy_data = False);
NOTICE:  created replication slot "vm1_test2_sub" on publisher
CREATE SUBSCRIPTION

postgres=# \dRs+                                                                                     List of subscriptions
     Name      |  Owner   | Enabled |   Publication   | Binary | Streaming | Two-phase commit | Disable on error | Synchronous commit |                   Conninfo                   | Skip LSN
---------------+----------+---------+-----------------+--------+-----------+------------------+------------------+--------------------+----------------------------------------------+----------
 vm1_test2_sub | postgres | t       | {vm2_test2_pub} | f      | f         | d                | f                | off                | host=10.0.1.xxx user=postgres dbname=postgres | 0/0
(1 row)
```

На ВМ2 подписываемся на таблицу test ВМ1:

```
postgres=# CREATE SUBSCRIPTION vm2_test1_sub CONNECTION 'host=10.0.1.xxx user=postgres dbname=postgres' PUBLICATION vm1_test_pub WITH (copy_data = False);
NOTICE:  created replication slot "vm2_test1_sub" on publisher
CREATE SUBSCRIPTION

postgres=# \dRs+ 
                                                                                     List of subscriptions
     Name      |  Owner   | Enabled |  Publication   | Binary | Streaming | Two-phase commit | Disable on error | Synchronous commit |                   Conninfo                    | Skip LSN
---------------+----------+---------+----------------+--------+-----------+------------------+------------------+--------------------+-----------------------------------------------+----------
 vm2_test1_sub | postgres | t       | {vm1_test_pub} | f      | f         | d                | f                | off                | host=10.0.1.xxx user=postgres dbname=postgres | 0/0
(1 row)
```

На ВМ3 подписываемся на таблицу test ВМ1 и таблицу test2 ВМ2:

```
postgres=# CREATE SUBSCRIPTION vm3_test1_sub CONNECTION 'host=10.0.1.xxx user=postgres dbname=postgres' PUBLICATION vm1_
test_pub WITH (copy_data = False);
NOTICE:  created replication slot "vm3_test1_sub" on publisher
CREATE SUBSCRIPTION
postgres=# CREATE SUBSCRIPTION vm3_test2_sub CONNECTION 'host=10.0.1.xxx user=postgres dbname=postgres' PUBLICATION vm2_t
est2_pub WITH (copy_data = False);
NOTICE:  created replication slot "vm3_test2_sub" on publisher
CREATE SUBSCRIPTION

postgres=# \dRs+ 
                                                                                      List of subscriptions
     Name      |  Owner   | Enabled |   Publication   | Binary | Streaming | Two-phase commit | Disable on error | Synchronous commit |                   Conninfo                    | Skip LSN
---------------+----------+---------+-----------------+--------+-----------+------------------+------------------+--------------------+-----------------------------------------------+----------
 vm3_test1_sub | postgres | t       | {vm1_test_pub}  | f      | f         | d                | f                | off                | host=10.0.1.xxx user=postgres dbname=postgres | 0/0
 vm3_test2_sub | postgres | t       | {vm2_test2_pub} | f      | f         | d                | f                | off                | host=10.0.1.xxx user=postgres dbname=postgres  | 0/0
(2 rows)

```

4. Проверяем.

На ВМ1 добавляем данные в таблицу test:

```
INSERT INTO test (id, title) VALUES (1, 'table test text 1');
INSERT INTO test (id, title) VALUES (2, 'table test text 2');
INSERT INTO test (id, title) VALUES (3, 'table test text 3');
INSERT INTO test (id, title) VALUES (4, 'table test text 4');
INSERT INTO test (id, title) VALUES (5, 'table test text 5');
```

На ВМ2 добавляем данные в таблицу test2:

```
INSERT INTO test2 (id, title) VALUES (1, 'table test2 text 6');
INSERT INTO test2 (id, title) VALUES (2, 'table test2 text 7');
INSERT INTO test2 (id, title) VALUES (3, 'table test2 text 8');
INSERT INTO test2 (id, title) VALUES (4, 'table test2 text 9');
INSERT INTO test2 (id, title) VALUES (5, 'table test2 text 10');
```

На ВМ1 селектим таблицу test2:

```
postgres=# SELECT * FROM test2;
 id |        title
----+---------------------
  1 | table test2 text 6
  2 | table test2 text 7
  3 | table test2 text 8
  4 | table test2 text 9
  5 | table test2 text 10
(5 rows)
```

На ВМ2 селектим таблицу test:

```
postgres=# SELECT * FROM test;
 id |       title
----+-------------------
  1 | table test text 1
  2 | table test text 2
  3 | table test text 3
  4 | table test text 4
  5 | table test text 5
(5 rows)
```

На ВМ3 селектим таблицу test и таблицу test2:

```
postgres=# postgres=# SELECT * FROM test;
 id |       title
----+-------------------
  1 | table test text 1
  2 | table test text 2
  3 | table test text 3
  4 | table test text 4
  5 | table test text 5
(5 rows)

postgres=# SELECT * FROM test2;
 id |        title
----+---------------------
  1 | table test2 text 6
  2 | table test2 text 7
  3 | table test2 text 8
  4 | table test2 text 9
  5 | table test2 text 10
(5 rows)
```

Репликация работает!















