- Создали тестовую БД Lego. Саму базу взяли отсюда: https://github.com/neondatabase-labs/postgres-sample-dbs?ysclid=mibjwjer85989084236
- Сделали запрос explain analyze select * from lego_sets where year = '1976';
- Получили ответ: Seq Scan on lego_sets  (cost=0.00..253.91 rows=68 width=42) (actual time=0.011..0.767 rows=68 loops=1)
  Filter: (year = 1976)
  Rows Removed by Filter: 11605
Planning Time: 0.050 ms
Execution Time: 0.780 ms
- Создали индекс в таблице lego_sets по столбцу year  create index lego_year on lego_sets(year);
- Запрос explain analyze select * from lego_sets where year = '1976';
- Вернул ответ:
  Bitmap Heap Scan on lego_sets  (cost=4.81..105.42 rows=68 width=42) (actual time=0.022..0.060 rows=68 loops=1)
  Recheck Cond: (year = 1976)
  Heap Blocks: exact=37
  ->  Bitmap Index Scan on lego_year  (cost=0.00..4.79 rows=68 width=0) (actual time=0.014..0.014 rows=68 loops=1)
        Index Cond: (year = 1976)
Planning Time: 0.052 ms
Execution Time: 0.074 ms
- По сравнению с запросом до создания индекса время выполнения уменьшилось в 10 раз с 0.780мс до 0.074мс
- Попробуем выполнить поиск по текстовому полю name. explain analyze select * from lego_sets where name ~ 'Space'; Получили результат Execution Time: 5.867 ms. План построился по seq scan таблицы.
- Попробуем выполнить поиск по gin индексу. Для этого создадим поле name_ts с типом данных ts_vector и заполним его данными из поля name. alter table lego_sets add column name_ts tsvector;   UPDATE lego_sets SET name_ts = to_tsvector('english', name);
- Создадим gin индекс по полю name_ts create index set_ts on lego_sets using gin(name_ts);
- Теперь при поиске по индексу select * from lego_sets where name_ts @@ to_tsquery('space'); получаем Bitmap Heap Scan on lego_sets  (cost=13.66..261.32 rows=113 width=94) (actual time=0.033..0.097 rows=113 loops=1)
  Recheck Cond: (name_ts @@ to_tsquery('space'::text))
  Heap Blocks: exact=61
  ->  Bitmap Index Scan on set_ts  (cost=0.00..13.63 rows=113 width=0) (actual time=0.022..0.022 rows=113 loops=1)
        Index Cond: (name_ts @@ to_tsquery('space'::text))
Planning Time: 0.118 ms
Execution Time: 0.114 ms
- Поиск пошел по индексу. Время выполнения уменьшилось в 50 раз.
- Дальше сделаем индекс на часть таблицы. create index year_index on lego_sets (year) where year < 2000;
- При выборке explain analyze select * from lego_sets where year = 1978 ; запрос пойдет по индексу   ->  Bitmap Index Scan on year_index  (cost=0.00..4.83 rows=73 width=0) (actual time=0.014..0.014 rows=73 loops=1)
- При выборке же explain analyze select * from lego_sets where year = 2000 ; условие в индекс не попадет и будет Seq Scan on lego_sets  (cost=0.00..439.91 rows=327 width=42) (actual time=0.102..0.841 rows=327 loops=1)
- Теперь создадим индекс по 2 полям create index year_index on lego_sets (theme_id, num_parts);
- Если в запросе фильтровать по theme_id или по theme_id и num_parts, то поиск идет по индексу.
- Если в запросе фильтровать только по 2 столбцу индекса num_parts, то по идее он должен идти мимо индекса, но происходит странная картина. Если в запросе указывать num_parts от 1 до 10, то планировщик использует seq scan explain analyze select * from lego_sets where num_parts = '1';  Seq Scan on lego_sets  (cost=0.00..439.91 rows=285 width=42) (actual time=0.064..0.868 rows=285 loops=1)
- Если же значение num_parts больше 10, то выборка идет по индексу. explain analyze select * from lego_sets where num_parts = '12'; Bitmap Heap Scan on lego_sets  (cost=203.86..415.80 rows=106 width=42) (actual time=0.143..0.213 rows=106 loops=1)
  Recheck Cond: (num_parts = 12)
  Heap Blocks: exact=70
  ->  Bitmap Index Scan on year_index  (cost=0.00..203.83 rows=106 width=0) (actual time=0.130..0.130 rows=106 loops=1)
        Index Cond: (num_parts = 12)
Planning Time: 0.056 ms
Execution Time: 0.231 ms
- К сожалению, не смог разобраться с этим поведением, когда поиск должен  идти по seq scan, но вместо этого идет по индексу. Использую версию postgres 18.
