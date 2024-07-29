# Секционирование таблицы

## Подготовка к выполнению ДЗ
Создана ВМ в VirtualBox с ОС Ubuntu 20.04. На машине установлен PostgreSQL 15.

## Выполнение ДЗ

0. Добавил тестовую базу данных.

```
wget https://edu.postgrespro.ru/demo_big.zip && sudo apt install unzip && unzip demo_big.zip

sudo -u postgres psql -d postgres -f /home/user/demo_big.sql -c 'alter database demo set search_path to bookings'
```

1. Найдем в базе самую большую таблицу:
	
```
SELECT nspname || '.' || relname AS "relation", pg_size_pretty(pg_relation_size(C.oid)) AS "size"
FROM pg_class C
LEFT JOIN pg_namespace N ON (N.oid = C.relnamespace)
WHERE nspname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_relation_size(C.oid) DESC;

relation                                          |size      |
--------------------------------------------------+----------+
bookings.ticket_flights                           |546 MB    |
bookings.boarding_passes                          |455 MB    |
bookings.tickets                                  |386 MB    |
bookings.ticket_flights_pkey                      |325 MB    |
```

По результату наша таблица bookings.ticket_flights.


2. Создадим секционированную таблицу part_ticket_flights по списку на основе ticket_flights

```
CREATE TABLE part_ticket_flights (like ticket_flights) PARTITION BY LIST (fare_conditions);
```

Создадим секции по типам билетов:

```
CREATE TABLE ticket_business PARTITION OF part_ticket_flights FOR VALUES IN ('Business');

CREATE TABLE ticket_comfort PARTITION OF part_ticket_flights FOR VALUES IN ('Comfort');

CREATE TABLE ticket_economy PARTITION OF part_ticket_flights FOR VALUES IN ('Economy');
```

3. Скопируем данные из ticket_flights в part_ticket_flights и посмотрим что получилось:

```
insert into part_ticket_flights
select * from ticket_flights;
```

Посмотрим сколько было записей в ticket_flights по типам билетов:

```
SELECT fare_conditions, COUNT(*)
FROM bookings.ticket_flights
GROUP BY fare_conditions
HAVING COUNT(*) > 1;

fare_conditions|count  |
---------------+-------+
Business       | 859656|
Comfort        | 139965|
Economy        |7392231|
```

Посмотрим каждую секцию в отдельности:

```
SELECT fare_conditions, COUNT(*)
FROM ticket_business
GROUP BY fare_conditions
HAVING COUNT(*) > 1;

fare_conditions|count |
---------------+------+
Business       |859656|
```

```
SELECT fare_conditions, COUNT(*)
FROM ticket_comfort
GROUP BY fare_conditions
HAVING COUNT(*) > 1;

fare_conditions|count |
---------------+------+
Comfort        |139965|
```

```
SELECT fare_conditions, COUNT(*)
FROM ticket_economy
GROUP BY fare_conditions
HAVING COUNT(*) > 1;

fare_conditions|count  |
---------------+-------+
Economy        |7392231|
```

Секционирование прошло успешно:)))















