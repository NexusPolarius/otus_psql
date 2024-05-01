# Настройка autovacuum с учетом особеностей производительности

## Подготовка к выполнению ДЗ
Создана ВМ в VirtualBox с ОС Ubuntu 20.04 с 2 ядрами и 4 Гб ОЗУ и SSD 10GB. Установлен PostgreSQL 15 с дефолтными настройками.

## Выполнение ДЗ
1. выполнить

> pgbench -i postgres

> pgbench -c8 -P 6 -T 60 -U postgres postgres

<img src="/HW05/xxx/1.PNG" alt="def_conf.png" /> 
	
2. Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
   
> autovacuum = on 
 
> autovacuum_vacuum_threshold = 100 
 
> autovacuum_analyze_threshold = 100
 
> autovacuum_vacuum_scale_factor = 0.5
 
> autovacuum_analyze_scale_factor = 0.2
 
> autovacuum_vacuum_cost_delay = 50
 
> autovacuum_vacuum_cost_limit = 500
    
3. Протестировать заново  
    
<img src="/HW05/xxx/2.PNG" alt="tun_conf.png" /> 

4. Что изменилось и почему?
   
Увеличилось tps и соответственно во время второго теста было совершено больше транзакций. Правильно настроеннный вакуум обеспечивает прирост производительности БД.

5. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк

> CREATE TABLE student(id serial, fio char(100));

> INSERT INTO student(fio) SELECT 'name1' FROM generate_series(1,1000000);

6. Посмотреть размер файла с таблицей

> SELECT pg_size_pretty(pg_total_relation_size('student'));

> 135 MB

7. 5 раз обновить все строчки и добавить к каждой строчке любой символ

> UPDATE student SET fio = 'name12';

> UPDATE student SET fio = 'name123';

> UPDATE student SET fio = 'name1234';

> UPDATE student SET fio = 'name12345';

> UPDATE student SET fio = 'name123456';

8. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум

> SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float AS "ratio%", last_autovacuum FROM pg_stat_user_tables WHERE relname = 'student';

> n_dead_tup =  4999944

9. Подождать некоторое время, проверяя, пришел ли автовакуум

> SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float AS "ratio%", last_autovacuum FROM pg_stat_user_tables WHERE relname = 'student';

> last_autovacuum = 2024-05-01 09:05:05.929488+03

> last_autovacuum = 2024-05-01 09:21:45.74986+03 - автовакуум пришел

> n_dead_tup =  0

10. 5 раз обновить все строчки и добавить к каждой строчке любой символ

> UPDATE student SET fio = 'name1234565';

> UPDATE student SET fio = 'name12345654';

> UPDATE student SET fio = 'name123456543';

> UPDATE student SET fio = 'name1234565432';

> UPDATE student SET fio = 'name12345654321';

11. Посмотреть размер файла с таблицей

> SELECT pg_size_pretty(pg_total_relation_size('student'));

> 808 MB

12. Отключить Автовакуум на конкретной таблице

> ALTER TABLE student SET (autovacuum_enabled = false);

13. 10 раз обновить все строчки и добавить к каждой строчке любой символ

> UPDATE student SET fio = 'name12';

> UPDATE student SET fio = 'name123';

> UPDATE student SET fio = 'name1234';

> UPDATE student SET fio = 'name12345';

> UPDATE student SET fio = 'name123456';

> UPDATE student SET fio = 'name1234565';

> UPDATE student SET fio = 'name12345654';

> UPDATE student SET fio = 'name123456543';

> UPDATE student SET fio = 'name1234565432';

> UPDATE student SET fio = 'name12345654321';

14. Посмотреть размер файла с таблицей

> SELECT pg_size_pretty(pg_total_relation_size('student'));

> 1925 MB

15. Объясните полученный результат

При отключенном автовакууме размер файла с таблицей будет постоянно расти т.к. не будут очищаться "мертвые строки". Когда "мертвые строки" очищаются из таблици автовакуумом, новые данные записываются сначала на освобожденные места тем самым физически файл с таблицей не растет пока есть освободившиеся место.

16. Не забудьте включить автовакуум)

> ALTER TABLE student SET (autovacuum_enabled = true);
