- Создадим схему my_schema create schema my_schema;  Сделаем ее дефолтной alter database test set search_path to my_schema, public;
- Создадим 2 таблицы CREATE TABLE table1 (id INTEGER PRIMARY KEY);   CREATE TABLE table2 (id INTEGER PRIMARY KEY); и заполним table1 данными INSERT INTO table1 (id) SELECT generate_series(1, 100);
- Создал каталог /var/lib/postgresql/backups/ под пользователем postgres <img width="642" height="102" alt="image" src="https://github.com/user-attachments/assets/99cd88f5-08ad-4aeb-b48d-553eb62b9754" />
- Скопировали таблицу table1 в backups test=# \copy table1 TO '/var/lib/postgresql/backups/table1.csv' CSV HEADER
- Скопировали файл в table2 \copy table2 FROM '/var/lib/postgresql/backups/table1.csv' CSV HEADER. Видим, что данные скопировались. 
   <img width="780" height="151" alt="image" src="https://github.com/user-attachments/assets/d3f2d689-d049-4127-92a7-f13c9d250415" />
- Далее попробуем то же самое сделать через дамп. Снимем дамп схемы my_schema pg_dump -U root -d test -n my_schema -Fc -f /var/lib/postgresql/backups/my_schema.dump
  <img width="1091" height="60" alt="image" src="https://github.com/user-attachments/assets/92101d63-e939-43ba-92b3-de5cd22a675e" />
- Дальше очистим данные в table2. truncate table table2;  Проверим, что данных в ней теперь нет. select count(*) from table2;
- Восстановим через pg_restore только таблицу table2 с ключом -a, чтобы восстановить только данные pg_restore -d test -h localhost -U postgres -a -t table2 /var/lib/postgresql/backups/my_schema.dump
- Проверим, что данные на месте. select count(*) from table2;
- <img width="1207" height="442" alt="image" src="https://github.com/user-attachments/assets/f0430ee2-b83a-4ac9-98af-e750546804f4" />




