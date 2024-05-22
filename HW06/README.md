# Механизм блокировок

## Подготовка к выполнению ДЗ
Создана ВМ в VirtualBox с ОС Ubuntu 20.04. Установлен PostgreSQL 15 с дефолтными настройками.

## Выполнение ДЗ

1. Настройте выполнение контрольной точки раз в 30 секунд.
    
```alter system set checkpoint_timeout = '30s';```

```sudo pg_ctlcluster 15 main restart```

```postgres=# select setting from pg_settings where name='checkpoint_timeout';
 setting
---------
 30
(1 row)
```

2. 10 минут c помощью утилиты pgbench подавайте нагрузку.

Перед тестом определяем значения Log Sequence Number и количество выполненных контрольных точек:

```postgres=# select pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn
---------------------------
 1/9F8BFF70
(1 row)
```

```postgres=# select checkpoints_timed from pg_stat_bgwriter;
 checkpoints_timed
-------------------
              6363
(1 row)
```

Запусткаем pgbench:

```pgbench -i postgres```
```pgbench -c 8 -P 60 -T 600 -U postgres postgres```

Данные после выполнения pgbench:

```postgres=# select pg_current_wal_insert_lsn();
 pg_current_wal_insert_lsn
---------------------------
 1/C0E0B288
(1 row)
```

```postgres=# select checkpoints_timed from pg_stat_bgwriter;
 checkpoints_timed
-------------------
             6383 
(1 row)
```

3. Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.

```SELECT pg_size_pretty('1/C0E0B288'::pg_lsn - '1/9F8BFF70'::pg_lsn) wal_size;
 wal_size
----------
 533 MB
(1 row)
```

В среднем на одну контрольную точку приходится 533/20 ~ 26 MB

4. Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?


```
tail -n 100 /var/log/postgresql/postgresql-15-main.log | grep checkpoint
2024-05-22 21:14:00.907 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:14:27.146 MSK [152170] LOG:  checkpoint complete: wrote 1687 buffers (10.3%); 0 WAL file(s) added, 1 removed, 0 recycled; write=26.183 s, sync=0.040 s, total=26.240 s; sync files=47, longest=0.009 s, average=0.001 s; distance=12792 kB, estimate=12792 kB
2024-05-22 21:14:30.146 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:14:57.089 MSK [152170] LOG:  checkpoint complete: wrote 2114 buffers (12.9%); 0 WAL file(s) added, 1 removed, 0 recycled; write=26.895 s, sync=0.032 s, total=26.943 s; sync files=17, longest=0.011 s, average=0.002 s; distance=25777 kB, estimate=25777 kB
2024-05-22 21:15:00.090 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:15:27.055 MSK [152170] LOG:  checkpoint complete: wrote 2014 buffers (12.3%); 0 WAL file(s) added, 2 removed, 0 recycled; write=26.904 s, sync=0.033 s, total=26.965 s; sync files=19, longest=0.008 s, average=0.002 s; distance=27872 kB, estimate=27872 kB
2024-05-22 21:15:30.058 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:15:57.091 MSK [152170] LOG:  checkpoint complete: wrote 2110 buffers (12.9%); 0 WAL file(s) added, 2 removed, 0 recycled; write=26.984 s, sync=0.013 s, total=27.034 s; sync files=8, longest=0.007 s, average=0.002 s; distance=27264 kB, estimate=27811 kB
2024-05-22 21:16:00.094 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:16:27.035 MSK [152170] LOG:  checkpoint complete: wrote 2224 buffers (13.6%); 0 WAL file(s) added, 1 removed, 0 recycled; write=26.891 s, sync=0.022 s, total=26.941 s; sync files=16, longest=0.010 s, average=0.002 s; distance=27266 kB, estimate=27757 kB
2024-05-22 21:16:30.038 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:16:57.089 MSK [152170] LOG:  checkpoint complete: wrote 2072 buffers (12.6%); 0 WAL file(s) added, 2 removed, 0 recycled; write=26.992 s, sync=0.019 s, total=27.052 s; sync files=9, longest=0.006 s, average=0.003 s; distance=26717 kB, estimate=27653 kB
2024-05-22 21:17:00.090 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:17:27.133 MSK [152170] LOG:  checkpoint complete: wrote 2202 buffers (13.4%); 0 WAL file(s) added, 2 removed, 0 recycled; write=26.989 s, sync=0.020 s, total=27.043 s; sync files=14, longest=0.006 s, average=0.002 s; distance=26382 kB, estimate=27526 kB
2024-05-22 21:17:30.134 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:17:57.059 MSK [152170] LOG:  checkpoint complete: wrote 2055 buffers (12.5%); 0 WAL file(s) added, 1 removed, 0 recycled; write=26.893 s, sync=0.017 s, total=26.925 s; sync files=8, longest=0.006 s, average=0.003 s; distance=25981 kB, estimate=27371 kB
2024-05-22 21:18:00.062 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:18:27.109 MSK [152170] LOG:  checkpoint complete: wrote 2197 buffers (13.4%); 0 WAL file(s) added, 2 removed, 0 recycled; write=26.992 s, sync=0.023 s, total=27.047 s; sync files=16, longest=0.007 s, average=0.002 s; distance=25900 kB, estimate=27224 kB
2024-05-22 21:18:30.110 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:18:57.038 MSK [152170] LOG:  checkpoint complete: wrote 2040 buffers (12.5%); 0 WAL file(s) added, 1 removed, 0 recycled; write=26.888 s, sync=0.017 s, total=26.928 s; sync files=8, longest=0.006 s, average=0.003 s; distance=25824 kB, estimate=27084 kB
2024-05-22 21:19:00.038 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:19:27.080 MSK [152170] LOG:  checkpoint complete: wrote 2187 buffers (13.3%); 0 WAL file(s) added, 2 removed, 0 recycled; write=26.990 s, sync=0.022 s, total=27.042 s; sync files=17, longest=0.008 s, average=0.002 s; distance=25972 kB, estimate=26973 kB
2024-05-22 21:19:30.082 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:19:57.033 MSK [152170] LOG:  checkpoint complete: wrote 2056 buffers (12.5%); 0 WAL file(s) added, 2 removed, 0 recycled; write=26.906 s, sync=0.010 s, total=26.951 s; sync files=8, longest=0.003 s, average=0.002 s; distance=26037 kB, estimate=26879 kB
2024-05-22 21:20:00.034 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:20:27.078 MSK [152170] LOG:  checkpoint complete: wrote 2182 buffers (13.3%); 0 WAL file(s) added, 1 removed, 0 recycled; write=26.996 s, sync=0.022 s, total=27.044 s; sync files=17, longest=0.005 s, average=0.002 s; distance=25638 kB, estimate=26755 kB
2024-05-22 21:20:30.082 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:20:57.043 MSK [152170] LOG:  checkpoint complete: wrote 2013 buffers (12.3%); 0 WAL file(s) added, 2 removed, 0 recycled; write=26.908 s, sync=0.017 s, total=26.962 s; sync files=9, longest=0.011 s, average=0.002 s; distance=25228 kB, estimate=26602 kB
2024-05-22 21:21:00.045 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:21:27.097 MSK [152170] LOG:  checkpoint complete: wrote 2023 buffers (12.3%); 0 WAL file(s) added, 1 removed, 0 recycled; write=26.996 s, sync=0.020 s, total=27.052 s; sync files=15, longest=0.008 s, average=0.002 s; distance=25697 kB, estimate=26512 kB
2024-05-22 21:21:30.098 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:21:57.043 MSK [152170] LOG:  checkpoint complete: wrote 2008 buffers (12.3%); 0 WAL file(s) added, 2 removed, 0 recycled; write=26.893 s, sync=0.014 s, total=26.946 s; sync files=9, longest=0.004 s, average=0.002 s; distance=25466 kB, estimate=26407 kB
2024-05-22 21:22:00.046 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:22:27.109 MSK [152170] LOG:  checkpoint complete: wrote 2466 buffers (15.1%); 0 WAL file(s) added, 1 removed, 0 recycled; write=27.010 s, sync=0.021 s, total=27.063 s; sync files=17, longest=0.005 s, average=0.002 s; distance=25376 kB, estimate=26304 kB
2024-05-22 21:22:30.110 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:22:57.050 MSK [152170] LOG:  checkpoint complete: wrote 2018 buffers (12.3%); 0 WAL file(s) added, 2 removed, 0 recycled; write=26.900 s, sync=0.018 s, total=26.940 s; sync files=10, longest=0.005 s, average=0.002 s; distance=25587 kB, estimate=26232 kB
2024-05-22 21:23:00.054 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:23:27.084 MSK [152170] LOG:  checkpoint complete: wrote 1993 buffers (12.2%); 0 WAL file(s) added, 1 removed, 0 recycled; write=26.988 s, sync=0.017 s, total=27.031 s; sync files=12, longest=0.005 s, average=0.002 s; distance=25626 kB, estimate=26172 kB
2024-05-22 21:23:30.086 MSK [152170] LOG:  checkpoint starting: time
2024-05-22 21:23:57.035 MSK [152170] LOG:  checkpoint complete: wrote 1978 buffers (12.1%); 0 WAL file(s) added, 2 removed, 0 recycled; write=26.900 s, sync=0.016 s, total=26.949 s; sync files=8, longest=0.007 s, average=0.002 s; distance=25556 kB, estimate=26110 kB
```

По логам видно что все контрольные точки выполнялись раз в 30 секунд. Так произошло потому что запись контрольных точек не превышала 27 секунд и во время тестирования журналы не разростались до значения max_wal_size которое было установлено в 1GB.

5. Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.

Синхронный режим:

```
postgres=# show synchronous_commit;
 synchronous_commit
--------------------
 on
(1 row)

postgres@psql:~$ pgbench -c8 -P 60 -T 600 postgres
pgbench (15.6 (Ubuntu 15.6-1.pgdg20.04+1))
starting vacuum...end.
progress: 60.0 s, 850.6 tps, lat 9.398 ms stddev 6.310, 0 failed
progress: 120.0 s, 828.7 tps, lat 9.653 ms stddev 6.417, 0 failed
progress: 180.0 s, 832.5 tps, lat 9.609 ms stddev 6.428, 0 failed
progress: 240.0 s, 814.5 tps, lat 9.821 ms stddev 6.544, 0 failed
progress: 300.0 s, 832.2 tps, lat 9.612 ms stddev 6.301, 0 failed
progress: 360.0 s, 824.8 tps, lat 9.698 ms stddev 6.735, 0 failed
progress: 420.0 s, 817.3 tps, lat 9.788 ms stddev 6.652, 0 failed
progress: 480.0 s, 813.9 tps, lat 9.828 ms stddev 6.453, 0 failed
progress: 540.0 s, 836.4 tps, lat 9.563 ms stddev 6.386, 0 failed
progress: 600.0 s, 828.9 tps, lat 9.650 ms stddev 6.408, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 496798
number of failed transactions: 0 (0.000%)
latency average = 9.660 ms
latency stddev = 6.465 ms
initial connection time = 26.656 ms
tps = 828.006553 (without initial connection time)
```

Асинхронный режим:

```
postgres=# show synchronous_commit;
 synchronous_commit
--------------------
 off
(1 row)

postgres@psql:~$ pgbench -c8 -P 60 -T 600 postgres
pgbench (15.6 (Ubuntu 15.6-1.pgdg20.04+1))
starting vacuum...end.
progress: 60.0 s, 1857.2 tps, lat 4.304 ms stddev 1.448, 0 failed
progress: 120.0 s, 1841.0 tps, lat 4.344 ms stddev 1.457, 0 failed
progress: 180.0 s, 1760.5 tps, lat 4.543 ms stddev 1.573, 0 failed
progress: 240.0 s, 1774.0 tps, lat 4.508 ms stddev 1.589, 0 failed
progress: 300.0 s, 1796.8 tps, lat 4.451 ms stddev 1.507, 0 failed
progress: 360.0 s, 1849.4 tps, lat 4.325 ms stddev 1.429, 0 failed
progress: 420.0 s, 1846.5 tps, lat 4.331 ms stddev 1.458, 0 failed
progress: 480.0 s, 1775.9 tps, lat 4.503 ms stddev 2.252, 0 failed
progress: 540.0 s, 1847.9 tps, lat 4.328 ms stddev 1.437, 0 failed
progress: 600.0 s, 1819.1 tps, lat 4.397 ms stddev 1.454, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 8
number of threads: 1
maximum number of tries: 1
duration: 600 s
number of transactions actually processed: 1090107
number of failed transactions: 0 (0.000%)
latency average = 4.402 ms
latency stddev = 1.578 ms
initial connection time = 25.116 ms
tps = 1816.874037 (without initial connection time)
```

В асинхронном режиме tps увеличелось в два раза. Включение синхронной записи защищает от возможной потери данных. Но, накладывает ограничение на пропускную способность сервера. Отключение синхронной запись обеспечит более высокую производительность по количеству транзакций.

6. Создайте новый кластер с включенной контрольной суммой страниц. Создайте таблицу. Вставьте несколько значений. Выключите кластер. Измените пару байт в таблице. Включите кластер и сделайте выборку из таблицы. Что и почему произошло? как проигнорировать ошибку и продолжить работу?

Создаем новый кластер:

```sudo pg_createcluster 15 main2 --start -- --data-checksums```

Создаем таблицу:

```CREATE TABLE test_tb(test_col text);```

Сделаем несколько записей:

```
INSERT INTO test_tb values ('zzz');
INSERT INTO test_tb values ('xxx');
INSERT INTO test_tb values ('ccc');
```

Посмотрим где лежит наша таблица:

```
postgres=# SELECT pg_relation_filepath('test_tb');
 pg_relation_filepath
----------------------
 base/5/16388
(1 row)
```

Выключим кластер, изменим несколько байт в таблице и включим кластер.

Сделаем выборку из таблицы:

```
postgres=# select * from test_tb;
WARNING:  page verification failed, calculated checksum 6076 but expected 8041
ERROR:  invalid page in block 0 of relation base/5/16388
```

Обнаружены поврежденные данные, чексумма не соответствет.

Что бы проигнорировать ошибку нужно включить ignore_checksum_failure:

```
postgres=# ALTER SYSTEM SET ignore_checksum_failure = on;
ALTER SYSTEM
postgres=# SELECT pg_reload_conf();
 pg_reload_conf
----------------
 t
(1 row)

postgres=# show ignore_checksum_failure;
 ignore_checksum_failure
-------------------------
 on
(1 row)

postgres=# select * from test_tbl;
WARNING:  page verification failed, calculated checksum 6076 but expected 8041
 test_column
-------------
 zzz
 xxx
 ccc
(3 rows)
```



