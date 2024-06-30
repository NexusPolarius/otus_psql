# Работа с join'ами, статистикой

## Подготовка к выполнению ДЗ
Создана ВМ в VirtualBox с ОС Ubuntu 20.04. На машине установлен PostgreSQL 15.

## Выполнение ДЗ

0. Добавил тестовую базу данных.

```
wget https://edu.postgrespro.ru/demo_small.zip && sudo apt install unzip && unzip demo_small.zip && sudo -u postgres psql -d postgres -f /home/user/demo_small.sql -c 'alter database demo set search_path to bookings'

 sudo -u postgres psql -d postgres -f /home/user/demo_small.sql -c 'alter database demo set search_path to bookings'
```

1. Реализовать прямое соединение двух или более таблиц

Беру таблицу с моделями самолетов:
	
```
demo=# select * from aircrafts;
 aircraft_code |        model        | range
---------------+---------------------+-------
 773           | Boeing 777-300      | 11100
 763           | Boeing 767-300      |  7900
 SU9           | Sukhoi SuperJet-100 |  3000
 320           | Airbus A320-200     |  5700
 321           | Airbus A321-200     |  5600
 319           | Airbus A319-100     |  6700
 733           | Boeing 737-300      |  4200
 CN1           | Cessna 208 Caravan  |  1200
 CR2           | Bombardier CRJ-200  |  2700
(9 rows)
```

И таблицу с местами:

```
demo=# select * from seats;
aircraft_code | seat_no | fare_conditions  
---------------+---------+-----------------
 319           | 2A      | Business
 319           | 2C      | Business
 319           | 2D      | Business
 319           | 2F      | Business
 319           | 3A      | Business
 319           | 3C      | Business
 319           | 3D      | Business
```

Хочу посмотреть какие места в каком самолете:

```
demo=# SELECT aircrafts.model, seats.seat_no, seats.fare_conditions
demo-# FROM seats
demo-# INNER JOIN aircrafts
demo-# ON seats.aircraft_code = aircrafts.aircraft_code;

        model        | seat_no | fare_conditions 
---------------------+---------+-----------------
 Airbus A319-100     | 2A      | Business
 Airbus A319-100     | 2C      | Business
 Airbus A319-100     | 2D      | Business
 Airbus A319-100     | 2F      | Business
 Airbus A319-100     | 3A      | Business
 Airbus A319-100     | 3C      | Business
 Airbus A319-100     | 3D      | Business
```

2. Реализовать левостороннее (или правостороннее) соединение двух или более таблиц

Беру таблицу с посадочными талонами:

```
demo=# select * from boarding_passes;
   ticket_no   | flight_id | boarding_no | seat_no
---------------+-----------+-------------+---------
 0005435212351 |     30625 |           1 | 2D
 0005435212386 |     30625 |           2 | 3G
 0005435212381 |     30625 |           3 | 4H
 0005432211370 |     30625 |           4 | 5D
 0005435212357 |     30625 |           5 | 11A
 0005435212360 |     30625 |           6 | 11E
 0005435212393 |     30625 |           7 | 11H
```

И таблицу с билетами:

```
demo=# select * from tickets;
 ticket_no    |book_ref|passenger_id|passenger_name      |contact_data                                                               
-------------+--------+------------+--------------------+---------------------------------------------------------------------------
0005432000987|06B046  |8149 604011 |VALERIY TIKHONOV    |{"phone": "+70127117011"}                                                  
0005432000988|06B046  |8499 420203 |EVGENIYA ALEKSEEVA  |{"phone": "+70378089255"}                                                  
0005432000989|E170C3  |1011 752484 |ARTUR GERASIMOV     |{"phone": "+70760429203"}                                                  
0005432000990|E170C3  |4849 400049 |ALINA VOLKOVA       |{"email": "volkova.alina_03101973@postgrespro.ru", "phone": "+70582584031"}
0005432000991|F313DD  |6615 976589 |MAKSIM ZHUKOV       |{"email": "m-zhukov061972@postgrespro.ru", "phone": "+70149562185"}        
```

Хочу получить информацию по посадочным и данным пассажира:

```
demo=# SELECT boarding_passes.boarding_no, boarding_passes.seat_no, tickets.passenger_name, tickets.contact_data
demo-# FROM boarding_passes
demo-# LEFT OUTER JOIN tickets
demo-# ON boarding_passes.ticket_no = tickets.ticket_no;

 boarding_no | seat_no |     passenger_name      |                                     contact_data
-------------+---------+-------------------------+---------------------------------------------------------------------------------------
           8 | 12E     | VERONIKA TARASOVA       | {"phone": "+70372550663"}
          20 | 17H     | MIKHAIL VLASOV          | {"phone": "+70702026741"}
          22 | 20B     | VIOLETTA SIDOROVA       | {"phone": "+70631420800"}
          32 | 26J     | ELIZAVETA KOROLEVA      | {"phone": "+70609911759"}
          52 | 33D     | LIDIYA ABRAMOVA         | {"phone": "+70218535523"}
          64 | 37E     | MATVEY FEDOTOV          | {"phone": "+70505998821"}
```

3. Реализовать кросс соединение двух или более таблиц

Возьму таблицу бронирования:

```
demo=# select * from bookings;
 book_ref |       book_date        | total_amount
----------+------------------------+--------------
 00000F   | 2016-09-02 02:12:00+03 |    265700.00
 000012   | 2016-09-11 08:02:00+03 |     37900.00
 000068   | 2016-10-13 13:27:00+03 |     18100.00
 000181   | 2016-10-08 12:28:00+03 |    131800.00
 0002D8   | 2016-10-05 20:40:00+03 |     23600.00
```

И таблицу билетов:

```
demo=# select * from tickets;
 ticket_no    |book_ref|passenger_id|passenger_name      |contact_data                                                               
-------------+--------+------------+--------------------+---------------------------------------------------------------------------
0005432000987|06B046  |8149 604011 |VALERIY TIKHONOV    |{"phone": "+70127117011"}                                                  
0005432000988|06B046  |8499 420203 |EVGENIYA ALEKSEEVA  |{"phone": "+70378089255"}                                                  
0005432000989|E170C3  |1011 752484 |ARTUR GERASIMOV     |{"phone": "+70760429203"}                                                  
0005432000990|E170C3  |4849 400049 |ALINA VOLKOVA       |{"email": "volkova.alina_03101973@postgrespro.ru", "phone": "+70582584031"}
0005432000991|F313DD  |6615 976589 |MAKSIM ZHUKOV       |{"email": "m-zhukov061972@postgrespro.ru", "phone": "+70149562185"} 
```

Посмотрим что из этого выйдет:

```
demo=# SELECT * FROM bookings CROSS JOIN tickets;
Killed
```

Долгое выполнение и в конце концов оно убилось)))))

Попробуем с ограничением:

```
demo=# SELECT * FROM bookings CROSS JOIN tickets limit 100;
book_ref |       book_date        | total_amount |   ticket_no   | book_ref | passenger_id |  passenger_name  |       contact_data
----------+------------------------+--------------+---------------+----------+--------------+------------------+---------------------------
 00000F   | 2016-09-02 02:12:00+03 |    265700.00 | 0005432000987 | 06B046   | 8149 604011  | VALERIY TIKHONOV | {"phone": "+70127117011"}
 000012   | 2016-09-11 08:02:00+03 |     37900.00 | 0005432000987 | 06B046   | 8149 604011  | VALERIY TIKHONOV | {"phone": "+70127117011"}
 000068   | 2016-10-13 13:27:00+03 |     18100.00 | 0005432000987 | 06B046   | 8149 604011  | VALERIY TIKHONOV | {"phone": "+70127117011"}
 000181   | 2016-10-08 12:28:00+03 |    131800.00 | 0005432000987 | 06B046   | 8149 604011  | VALERIY TIKHONOV | {"phone": "+70127117011"}
 0002D8   | 2016-10-05 20:40:00+03 |     23600.00 | 0005432000987 | 06B046   | 8149 604011  | VALERIY TIKHONOV | {"phone": "+70127117011"}
 0002DB   | 2016-09-26 05:30:00+03 |    101500.00 | 0005432000987 | 06B046   | 8149 604011  | VALERIY TIKHONOV | {"phone": "+70127117011"}
 0002E0   | 2016-09-08 15:09:00+03 |     89600.00 | 0005432000987 | 06B046   | 8149 604011  | VALERIY TIKHONOV | {"phone": "+70127117011"}
 0002F3   | 2016-09-07 04:31:00+03 |     69600.00 | 0005432000987 | 06B046   | 8149 604011  | VALERIY TIKHONOV | {"phone": "+70127117011"}
 00034E   | 2016-10-02 15:52:00+03 |     73300.00 | 0005432000987 | 06B046   | 8149 604011  | VALERIY TIKHONOV | {"phone": "+70127117011"}
 000352   | 2016-09-03 01:02:00+03 |    109500.00 | 0005432000987 | 06B046   | 8149 604011  | VALERIY TIKHONOV | {"phone": "+70127117011"}
 000374   | 2016-10-10 09:13:00+03 |    136200.00 | 0005432000987 | 06B046   | 8149 604011  | VALERIY TIKHONOV | {"phone": "+70127117011"}
 00044D   | 2016-09-26 23:24:00+03 |      6000.00 | 0005432000987 | 06B046   | 8149 604011  | VALERIY TIKHONOV | {"phone": "+70127117011"}
```

4. Реализовать полное соединение двух или более таблиц

Беру таблицу с аэропортами:

```
demo=# select * from airports ;
 airport_code |     airport_name     |           city           |    longitude    |    latitude     |      timezone
--------------+----------------------+--------------------------+-----------------+-----------------+--------------------
 MJZ          | Мирный               | Мирный                   |      114.038928 |       62.534689 | Asia/Yakutsk
 NBC          | Бегишево             | Нижнекамск               |           52.06 |           55.34 | Europe/Moscow
 NOZ          | Спиченково           | Новокузнецк              |         86.8772 |         53.8114 | Asia/Novokuznetsk
 NAL          | Нальчик              | Нальчик                  |         43.6366 |         43.5129 | Europe/Moscow
 OGZ          | Беслан               | Владикавказ              |         44.6066 |         43.2051 | Europe/Moscow
 CSY          | Чебоксары            | Чебоксары                |         47.3473 |         56.0903 | Europe/Moscow
```

И таблицу с рейсами:

```
demo=# select * from flights;
 flight_id | flight_no |  scheduled_departure   |   scheduled_arrival    | departure_airport | arrival_airport |  status   | aircraft_code |    actual_departure    |     actual_arrival
-----------+-----------+------------------------+------------------------+-------------------+-----------------+-----------+---------------+------------------------+------------------------
         1 | PG0405    | 2016-09-13 08:35:00+03 | 2016-09-13 09:30:00+03 | DME               | LED             | Arrived   | 321           | 2016-09-13 08:44:00+03 | 2016-09-13 09:39:00+03
         2 | PG0404    | 2016-10-03 18:05:00+03 | 2016-10-03 19:00:00+03 | DME               | LED             | Arrived   | 321           | 2016-10-03 18:06:00+03 | 2016-10-03 19:01:00+03
         3 | PG0405    | 2016-10-03 08:35:00+03 | 2016-10-03 09:30:00+03 | DME               | LED             | Arrived   | 321           | 2016-10-03 08:39:00+03 | 2016-10-03 09:34:00+03
```

Хочу посмотреть вылеты по всем городам:

```
demo=# SELECT airports.city, flights.flight_no, flights.scheduled_departure
demo-# FROM flights
demo-# FULL OUTER JOIN airports
demo-# ON flights.departure_airport = airports.airport_code;

           city           | flight_no |  scheduled_departure
--------------------------+-----------+------------------------
 Москва                   | PG0405    | 2016-09-13 08:35:00+03
 Москва                   | PG0404    | 2016-10-03 18:05:00+03
 Москва                   | PG0405    | 2016-10-03 08:35:00+03
 Москва                   | PG0402    | 2016-11-07 11:25:00+03
 Москва                   | PG0405    | 2016-10-14 08:35:00+03
 Москва                   | PG0404    | 2016-10-14 18:05:00+03
 Москва                   | PG0403    | 2016-10-14 10:25:00+03
 Москва                   | PG0402    | 2016-10-14 11:25:00+03
```

5. Реализовать запрос, в котором будут использованы разные типы соединений

Хочу посмотреть информацию по брони на билеты бизнесс класса: 

```
demo=# SELECT tickets.ticket_no, tickets.book_ref, bookings.book_date, bookings.total_amount, ticket_flights.fare_conditions
demo-# FROM tickets
demo-# INNER JOIN bookings ON tickets.book_ref = bookings.book_ref
demo-# LEFT JOIN ticket_flights ON tickets.ticket_no = ticket_flights.ticket_no
demo-# WHERE ticket_flights.fare_conditions = 'Business';
   ticket_no   | book_ref |       book_date        | total_amount | fare_conditions
---------------+----------+------------------------+--------------+-----------------
 0005435838975 | 00000F   | 2016-09-02 02:12:00+03 |    265700.00 | Business
 0005433986734 | 0002DB   | 2016-09-26 05:30:00+03 |    101500.00 | Business
 0005434407173 | 0002E0   | 2016-09-08 15:09:00+03 |     89600.00 | Business
 0005435653907 | 00034E   | 2016-10-02 15:52:00+03 |     73300.00 | Business
 0005433342103 | 000352   | 2016-09-03 01:02:00+03 |    109500.00 | Business
 0005433342102 | 000352   | 2016-09-03 01:02:00+03 |    109500.00 | Business
 0005433986060 | 00044E   | 2016-09-14 04:39:00+03 |    140100.00 | Business
 0005433986060 | 00044E   | 2016-09-14 04:39:00+03 |    140100.00 | Business
 0005433986059 | 00044E   | 2016-09-14 04:39:00+03 |    140100.00 | Business
 0005433101280 | 000511   | 2016-08-29 02:40:00+03 |     26700.00 | Business
```


















