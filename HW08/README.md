# Нагрузочное тестирование и тюнинг PostgreSQL

## Подготовка к выполнению ДЗ
Создана ВМ в VirtualBox с ОС Ubuntu 20.04. Установлен PostgreSQL 15 с дефолтными настройками и создана БД tune_db.

## Выполнение ДЗ

1. Выполним замер pgbench с дефолтными настройками Postgres.
	
```postgres@psql:~$ pgbench -i tune_db```
	
```postgres@psql:~$ pgbench -c 50 -j 2 -P 10 -T 60 tune_db```
  
```
pgbench (15.6 (Ubuntu 15.6-1.pgdg20.04+1))
starting vacuum...end.
progress: 10.0 s, 746.4 tps, lat 66.096 ms stddev 54.154, 0 failed
progress: 20.0 s, 803.0 tps, lat 62.163 ms stddev 49.751, 0 failed
progress: 30.0 s, 795.8 tps, lat 62.824 ms stddev 52.874, 0 failed
progress: 40.0 s, 811.6 tps, lat 61.639 ms stddev 49.462, 0 failed
progress: 50.0 s, 780.5 tps, lat 64.088 ms stddev 54.147, 0 failed
progress: 60.0 s, 792.0 tps, lat 63.136 ms stddev 51.439, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 47343
number of failed transactions: 0 (0.000%)
latency average = 63.325 ms
latency stddev = 51.998 ms
initial connection time = 90.743 ms
tps = 788.857098 (without initial connection time)
```

2. С помощью ресурса https://www.pgconfig.org посмотрим рекомендованные настройки для нашего стэнда.

```
# Generated by PGConfig 3.1.4 (1fe6d98dedcaad1d0a114617cfd08b4fed1d8a01)
# https://api.pgconfig.org/v1/tuning/get-config?format=conf&&log_format=csvlog&max_connections=100&pg_version=15&environment_name=Mixed&total_ram=4GB&cpus=2&drive_type=SSD&arch=x86-64&os_type=linux

# Memory Configuration
shared_buffers = 512MB
effective_cache_size = 2GB
work_mem = 4MB
maintenance_work_mem = 102MB

# Checkpoint Related Configuration
min_wal_size = 2GB
max_wal_size = 3GB
checkpoint_completion_target = 0.9
wal_buffers = -1

# Network Related Configuration
listen_addresses = '*'
max_connections = 100

# Storage Configuration
random_page_cost = 1.1
effective_io_concurrency = 200

# Worker Processes Configuration
max_worker_processes = 8
max_parallel_workers_per_gather = 2
max_parallel_workers = 2
```

3. Создадим конфигурационный файл в include_dir = 'conf.d' опираясь на настройки полученные выше, но выкрутив некоторые параметры для большей производительности не обращая внимание на возможные проблемы с надежностью.

```sudo nano /etc/postgresql/15/main/conf.d/tune.conf```


```
# Memory Configuration
shared_buffers = 2GB
effective_cache_size = 2GB
work_mem = 16MB
maintenance_work_mem = 500MB

# Checkpoint Related Configuration
min_wal_size = 2GB
max_wal_size = 5GB
checkpoint_completion_target = 0.9
wal_buffers = -1
checkpoint_timeout = 30min

# Network Related Configuration
listen_addresses = '*'
max_connections = 100

# Storage Configuration
random_page_cost = 1.1
effective_io_concurrency = 200

# Worker Processes Configuration
max_worker_processes = 8
max_parallel_workers_per_gather = 2
max_parallel_workers = 2

# more
fsync = off
full_page_writes = off
synchronous_commit = off
```

> shared_buffers - Используется для кэширования данных. По умолчанию низкое значение (для поддержки как можно большего кол-ва ОС). Начать стоит с его изменения.
> Согласно документации, рекомендуемое значение для данного параметра - 25% от общей оперативной памяти на сервере.

> work_mem - устанавливает базовый максимальный объем памяти, который будет использоваться операцией запроса (например, сортировкой или хэш-таблицей)
> перед записью во временные файлы на диске.

> Maintenance_work_mem - указывает максимальный объем памяти, который будет использоваться операциями обслуживания, такими как
> VACUUM, CREATE INDEX и ALTER TABLE ADD FOREIGN KEY.

> max_wal_size -  максимальный размер, до которого может вырастать WAL между автоматическими контрольными точками в WAL.
> Значение по умолчанию — 1 ГБ. Увеличение этого параметра может привести к увеличению времени, которое потребуется для восстановления после сбоя,
> но позволяет реже выполнять операцию сбрасывания на диск.

>  checkpoint_timeout - максимальное время между автоматическими контрольными точками в WAL (в секундах). Допускаются значения от 30 секунд до одного дня.

> fsync - отключение fsync часто даёт выигрыш в скорости, это может привести к неисправимой порче данных в случае отключения питания или сбоя системы.
> Поэтому отключать fsync рекомендуется, только если вы легко сможете восстановить всю базу из внешнего источника.

> full_page_writes - отключение этого параметра ускоряет обычные операции, но может привести к неисправимому повреждению или незаметной порче данных после сбоя системы.
> Так как при этом возникают практически те же риски, что и при отключении fsync, хотя и в меньшей степени, отключать его следует только при тех же обстоятельствах,
> которые перечислялись в рекомендациях для вышеописанного параметра.

> synchronous_commit - в отличие от fsync, значение off этого параметра не угрожает целостности данных: сбой операционной системы или базы данных может привести
> к потере последних транзакций, считавшихся зафиксированными, но состояние базы данных будет точно таким же, как и в случае штатного прерывания этих транзакций.
> Поэтому выключение режима synchronous_commit может быть полезной альтернативой отключению fsync, когда производительность важнее, чем надёжная гарантия
> сохранности каждой транзакции.

Перезапустим кластер

```sudo pg_ctlcluster 15 main restart```


4. Выполним снова замер pgbench с тюнингованными настройками Postgres.
	
```postgres@psql:~$ pgbench -c 50 -j 2 -P 10 -T 60 tune_db```
  
```
pgbench (15.6 (Ubuntu 15.6-1.pgdg20.04+1))
starting vacuum...end.
progress: 10.0 s, 1504.9 tps, lat 32.822 ms stddev 18.157, 0 failed
progress: 20.0 s, 1557.7 tps, lat 32.086 ms stddev 16.965, 0 failed
progress: 30.0 s, 1566.0 tps, lat 31.935 ms stddev 16.865, 0 failed
progress: 40.0 s, 1560.3 tps, lat 32.043 ms stddev 17.140, 0 failed
progress: 50.0 s, 1571.0 tps, lat 31.791 ms stddev 16.423, 0 failed
progress: 60.0 s, 1569.5 tps, lat 31.882 ms stddev 18.291, 0 failed
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 1
query mode: simple
number of clients: 50
number of threads: 2
maximum number of tries: 1
duration: 60 s
number of transactions actually processed: 93344
number of failed transactions: 0 (0.000%)
latency average = 32.108 ms
latency stddev = 17.343 ms
initial connection time = 102.175 ms
tps = 1555.862703 (without initial connection time)
```

Как мы видим наш тюнинг удался. Значения latency уменьшились, а tps увеличился.











