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

-- Добавление столбца tsvector с автоматическим заполнением для полнотекстового поиска

postgres=# ALTER TABLE articles ADD COLUMN content_tsvector TSVECTOR GENERATED ALWAYS AS (to_tsvector('english', content)) STORED;
ALTER TABLE
```

1. Создать индекс к какой-либо из таблиц вашей БД

Создал индекс по числовому столбцу:
	
```
postgres=# CREATE INDEX ON index_test_table(random_num);
CREATE INDEX
```

Выполнил план запроса для выборки по точному значению:

```
postgres=# EXPLAIN SELECT * FROM index_test_table WHERE random_num = 100;
                                               QUERY PLAN
--------------------------------------------------------------------------------------------------------
 Index Scan using index_test_table_random_num_idx on index_test_table  (cost=0.29..8.31 rows=1 width=7)
   Index Cond: (random_num = 100)
(2 rows)
```

В результате выполнения EXPLAIN видим "Index Scan", что говорит о использовании индекса.

2. Реализовать индекс для полнотекстового поиска

Создаю GIN индекс для столбца tsvector

```
postgres=# CREATE INDEX idx_articles_content_tsvector ON articles USING gin(content_tsvector);
CREATE INDEX
```

Так как в тестовой таблице мало данных EXPLAIN будет выполнятся без использования индекса, поэтому на время отключем Seq Scan:

```
postgres=# SET enable_seqscan = OFF;
SET
```

Выполним план запроса с полнотекстовым поиском

```
postgres=# EXPLAIN SELECT title, content FROM articles WHERE content_tsvector @@ to_tsquery('english', 'PostgreSQL & full-text');
                                                  QUERY PLAN
---------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on articles  (cost=20.00..24.01 rows=1 width=64)
   Recheck Cond: (content_tsvector @@ '''postgresql'' & ''full-text'' <-> ''full'' <-> ''text'''::tsquery)
   ->  Bitmap Index Scan on idx_articles_content_tsvector  (cost=0.00..20.00 rows=1 width=0)
         Index Cond: (content_tsvector @@ '''postgresql'' & ''full-text'' <-> ''full'' <-> ''text'''::tsquery)
(4 rows)
```

В результате выполнения EXPLAIN видим "Index Scan", что говорит о использовании индекса.

Включаем Seq Scan:

```
postgres=# SET enable_seqscan = ON;
SET
```

3. Реализовать индекс на часть таблицы или индекс на поле с функцией

Создал индекс на часть таблицы, где значения столбца bool_field = FALSE:

```
postgres=# CREATE INDEX ON index_test_table(bool_field) WHERE bool_field = FALSE;
CREATE INDEX
```

Выполним план запроса, где bool_field = TRUE и видим что используется Seq Scan:

```
postgres=# EXPLAIN SELECT * FROM index_test_table WHERE bool_field = TRUE;
                                      QUERY PLAN
--------------------------------------------------------------------------------------
 Seq Scan on index_test_table  (cost=10000000000.00..10000001443.00 rows=967 width=7)
   Filter: bool_field
 JIT:
   Functions: 2
   Options: Inlining true, Optimization true, Expressions true, Deforming true
(5 rows)
```

Выполним план запроса, где bool_field = FALSE и видим что используется Index Scan:

```
postgres=# EXPLAIN SELECT * FROM index_test_table WHERE bool_field = FALSE;
                                             QUERY PLAN
----------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on index_test_table  (cost=864.22..2297.55 rows=99033 width=7)
   Recheck Cond: (NOT bool_field)
   ->  Bitmap Index Scan on index_test_table_bool_field_idx  (cost=0.00..839.46 rows=99033 width=0)
(3 rows)
```

4. Создать индекс на несколько полей

СОздал индекс по двум колонкам

```
postgres=# CREATE INDEX ON index_test_table(random_num, random_text);
CREATE INDEX
```

Выполнил план запроса и вижу что используется Index Scan:

```
postgres=# EXPLAIN SELECT * FROM index_test_table WHERE random_num <= 100 AND random_text = 'a';
                                                     QUERY PLAN
--------------------------------------------------------------------------------------------------------------------
 Index Scan using index_test_table_random_num_random_text_idx on index_test_table  (cost=0.29..9.38 rows=1 width=7)
   Index Cond: ((random_num <= 100) AND (random_text = 'a'::text))
(2 rows)
```

















