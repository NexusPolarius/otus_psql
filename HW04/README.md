# Работа с базами данных, пользователями и правами

## Подготовка к выполнению ДЗ
Создана ВМ в VirtualBox с ОС Ubuntu 20.04, в которой создан кластер postgresql-14.

## Выполнение ДЗ
1. создайте новый кластер PostgresSQL 14

2. зайдите в созданный кластер под пользователем postgres

> sudo su postgres
> psql

3. создайте новую базу данных testdb

> CREATE DATABASE testdb;

4. зайдите в созданную базу данных под пользователем postgres

> \c testdb

5. создайте новую схему testnm

> CREATE SCHEMA testnm;

6. создайте новую таблицу t1 с одной колонкой c1 типа integer

> CREATE TABLE t1(c1 integer);

7. вставьте строку со значением c1=1

> INSERT INTO t1 values(1);

8. создайте новую роль readonly

> CREATE role readonly;

9. дайте новой роли право на подключение к базе данных testdb

> grant connect on DATABASE testdb TO readonly;

10. дайте новой роли право на использование схемы testnm

> grant usage on SCHEMA testnm to readonly;

11. дайте новой роли право на select для всех таблиц схемы testnm

> grant SELECT on all TABLEs in SCHEMA testnm TO readonly;

12. создайте пользователя testread с паролем test123

> CREATE USER testread with password 'test123';

13. дайте роль readonly пользователю testread

> grant readonly TO testread;

14. зайдите под пользователем testread в базу данных testdb

> \c testdb testread

15. сделайте select * from t1;

> SELECT * FROM t1;

16. получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)

> не получилось т.к. у роли readonly даны права только на схему testnm, а таблица t1 создалась в схеме public.

17. напишите что именно произошло в тексте домашнего задания

18. у вас есть идеи почему? ведь права то дали?

19. посмотрите на список таблиц

20. подсказка в шпаргалке под пунктом 20

21. а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)

> потому что в search_path скорее всего "$user", public при том что схемы $USER нет то таблица по умолчанию создалась в public

22. вернитесь в базу данных testdb под пользователем postgres

> \c testdb postgres

23. удалите таблицу t1

> DROP TABLE t1;

24. создайте ее заново но уже с явным указанием имени схемы testnm

> CREATE TABLE testnm.t1(c1 integer);

25. вставьте строку со значением c1=1

> INSERT INTO testnm.t1 values(1);

26. зайдите под пользователем testread в базу данных testdb

> \c testdb testread

27. сделайте select * from testnm.t1;

> SELECT * FROM testnm.t1;

28. получилось?

> не получилось, потому что grant SELECT on all TABLEs in SCHEMA testnm TO readonly дал доступ только для существующих на тот момент времени таблиц а t1 пересоздавалась


29. есть идеи почему? если нет - смотрите шпаргалку

30. как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку

> \c testdb postgres; 
> ALTER default privileges in SCHEMA testnm grant SELECT on TABLES to readonly; 
> \c testdb testread;

31. сделайте select * from testnm.t1;

> select * from testnm.t1;

32. получилось?

> не получилось, потому что ALTER default будет действовать для новых таблиц а grant SELECT on all TABLEs in SCHEMA testnm TO readonly отработал только для существующих на тот момент времени. надо сделать снова или grant SELECT или пересоздать таблицу

33. есть идеи почему? если нет - смотрите шпаргалку

34. сделайте select * from testnm.t1;

35. получилось?

> да, после grant SELECT on all TABLEs in SCHEMA testnm TO readonly; получилось.

36. ура!

37. теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);

38. а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?

> это все потому что search_path указывает в первую очередь на схему public. 
> А схема public создается в каждой базе данных по умолчанию. 
> И grant на все действия в этой схеме дается роли public. 
> А роль public добавляется всем новым пользователям. 
> Соответсвенно каждый пользователь может по умолчанию создавать объекты в схеме public любой базы данных, 
> ес-но если у него есть право на подключение к этой базе данных. 

39. есть идеи как убрать эти права? если нет - смотрите шпаргалку

> Чтобы раз и навсегда забыть про роль public - а в продакшн базе данных про нее лучше забыть - выполните следующие действия 
> \c testdb postgres; 
> REVOKE CREATE on SCHEMA public FROM public; 
> REVOKE ALL on DATABASE testdb FROM public; 
> \c testdb testread; 

40. если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды

> REVOKE CREATE on SCHEMA public FROM public; - запрещает роли public создавать объекты в схеме public

> REVOKE ALL on DATABASE testdb FROM public; - запрет роли public делать что либо с базой testdb

41. теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);

42. расскажите что получилось и почему

Из-за выполненных выше запретов пользователь testread не может создавать объекты в схеме public.


