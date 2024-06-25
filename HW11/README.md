# Работа с индексами

## Подготовка к выполнению ДЗ
Создана ВМ в VirtualBox с ОС Ubuntu 20.04. На машине установлен PostgreSQL 15.

## Выполнение ДЗ

0. Создал тестовые таблицы и заполнил их данными.

```
postgres=# CREATE TABLE index_test_table( random_num INTEGER, random_text TEXT, bool_field BOOLEAN );
CREATE TABLE

postgres=# INSERT INTO index_test_table(random_num, random_text, bool_field) SELECT s.id, chr((32 + random() * 94)::INTEGER),
     random() < 0.01 FROM generate_series(1, 100000) AS s(id) ORDER BY random();
INSERT 0 100000
```

```
postgres=# CREATE TABLE articles ( id SERIAL PRIMARY KEY, title TEXT, content TEXT );
CREATE TABLE

postgres=# INSERT INTO articles (title, content) VALUES ('PostgreSQL Tutorial', 'This tutorial covers the basics of PostgreSQL.'),
 ('Full-Text Search in PostgreSQL', 'Learn how to use full-text search in PostgreSQL.'),
 ('Advanced PostgreSQL Features', 'Explore advanced features of PostgreSQL, including GIN indexes and full-text search.');
INSERT 0 3

postgres=# ALTER TABLE articles ADD COLUMN content_tsvector TSVECTOR GENERATED ALWAYS AS (to_tsvector('english', content)) STORED;
ALTER TABLE
```

1. Создать индекс к какой-либо из таблиц вашей БД
	
```
postgres=# CREATE TABLE test (id SERIAL PRIMARY KEY, title TEXT);
CREATE TABLE

postgres=# CREATE TABLE test2 (id SERIAL PRIMARY KEY, title TEXT);
CREATE TABLE
```

2. Реализовать индекс для полнотекстового поиска

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

3. Реализовать индекс на часть таблицы или индекс на поле с функцией

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

4. Создать индекс на несколько полей

```
INSERT INTO test (id, title) VALUES (1, 'table test text 1');
INSERT INTO test (id, title) VALUES (2, 'table test text 2');
INSERT INTO test (id, title) VALUES (3, 'table test text 3');
INSERT INTO test (id, title) VALUES (4, 'table test text 4');
INSERT INTO test (id, title) VALUES (5, 'table test text 5');
```

















