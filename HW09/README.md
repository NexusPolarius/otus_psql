# Бэкапы

## Подготовка к выполнению ДЗ
Создана ВМ в VirtualBox с ОС Ubuntu 20.04. Установлен PostgreSQL 15 с дефолтными настройками.

## Выполнение ДЗ

1. Создаем БД, схему и в ней таблицу.
	
```
postgres=# create database test_db;
CREATE DATABASE

postgres=# \c test_db
You are now connected to database "test_db" as user "postgres".

test_db=# create schema test;
CREATE SCHEMA
```

2. Заполним таблицы автосгенерированными 100 записями.

```
test_db=# create table test.test_tb as select generate_series(1, 100) as id, md5(random()::text)::char(10) as text;
SELECT 100
```

3. Под линукс пользователем Postgres создадим каталог для бэкапов.

```
user@psql:~$ sudo mkdir /opt/backups
user@psql:~$ sudo chown postgres:postgres /opt/backups/
```

4. Сделаем логический бэкап используя утилиту COPY.

```
test_db=# \copy test.test_tb to '/opt/backups/tb_backup.sql'
COPY 100
```

5. Восстановим в 2 таблицу данные из бэкапа.

```
test_db=# create table test.test_tb2 (id int, text char(10));
CREATE TABLE

test_db=# \copy test.test_tb2 from '/opt/backups/tb_backup.sql'
COPY 100

test_db=# select * from test.test_tb2;

 id  |    text
-----+------------
   1 | 8d1e3de36a
   2 | 0156d31e93
   3 | bd82594555
   4 | 3d263afcac
   
  98 | 2ed74a0683
  99 | 5016c8a14e
 100 | 7e76919574
(100 rows)   
```
	
6. Используя утилиту pg_dump создадим бэкап в кастомном сжатом формате двух таблиц.

```user@psql:~$ sudo -u postgres pg_dump -d test_db -U postgres --format=d --table=test.test_t* -Z 5 -f /opt/backups/test_tb6.sql```

7. Используя утилиту pg_restore восстановим в новую БД только вторую таблицу!

```
postgres=# create database test_db2;
CREATE DATABASE

postgres=# \c test_db2
You are now connected to database "test_db2" as user "postgres".

test_db2=# create schema test;
CREATE SCHEMA
```

```sudo -u postgres pg_restore -d test_db2 -U postgres -n test -t test_tb2 /opt/backups/test_tb6.sql```

```test_db2=# SELECT table_name FROM information_schema.tables WHERE table_schema='test';
 table_name
------------
 test_tb2
(1 row)
```













