- Выбрали для секционирования таблицу bookings, так как в ней 1300000 строк, и ее довольно удобно можно разбить по столбцу book_date месяцам.<img width="492" height="102" alt="image" src="https://github.com/user-attachments/assets/f19ac001-df06-4bc7-bae3-c0a47ea93784" />
- Переименуем таблицу bookings в bookings_old  ALTER TABLE bookings RENAME TO bookings_old;
- Создал секционированную таблицу bookings \
  CREATE TABLE bookings ( \
    book_ref char(6) NOT NULL, \
    book_date timestamptz NOT NULL, \
    total_amount numeric(10,2) NOT NULL, \
    PRIMARY KEY (book_ref, book_date) \
) PARTITION BY RANGE (book_date);

- Создал партиции с разбивкой по месяцам \
    CREATE TABLE bookings_2025_09 PARTITION OF bookings
    FOR VALUES FROM ('2025-09-01') TO ('2025-10-01');

    CREATE TABLE bookings_2025_10 PARTITION OF bookings
    FOR VALUES FROM ('2025-10-01') TO ('2025-11-01');

    CREATE TABLE bookings_2025_11 PARTITION OF bookings
    FOR VALUES FROM ('2025-11-01') TO ('2025-12-01');

    CREATE TABLE bookings_2025_12 PARTITION OF bookings
    FOR VALUES FROM ('2025-12-01') TO ('2026-01-01'); \
  <img width="303" height="288" alt="image" src="https://github.com/user-attachments/assets/ee87e383-2ed0-4145-a6b6-e8da726f5ccd" />

- Перенес данные из bookings_old в bookings  INSERT INTO bookings (book_ref, book_date, total_amount)
SELECT book_ref, book_date, total_amount FROM bookings_old;
- Сравним скорость выполнения запросов в bookings_old и bookings. 
-  explain analyze select * from bookings_old where book_date > '2025-12-01'; <img width="866" height="189" alt="image" src="https://github.com/user-attachments/assets/df816324-b154-4dc1-8532-c9639e311ae6" />
- explain analyze select * from bookings where book_date > '2025-12-01'; <img width="870" height="103" alt="image" src="https://github.com/user-attachments/assets/65a7e1f0-ac47-44ca-a23a-d565773194da" />
- Проверим операции над таблицей: update bookings set total_amount = '3000' where book_ref = 'PWH5TL'; \
   select * from bookings where book_date > '2025-12-01' and book_ref = 'PWH5TL'; \
  <img width="499" height="58" alt="image" src="https://github.com/user-attachments/assets/32850a48-5339-4fe8-a60d-0fd86395296c" /> \
  Добавим строку insert into bookings values('PWWDDD', '2025-12-01 00:00:43.875 +0300', '5000'); \
                 select * from bookings where book_date > '2025-12-01' and book_ref = 'PWWDDD'; \
                 <img width="509" height="61" alt="image" src="https://github.com/user-attachments/assets/5d4dd49b-fab3-4cc7-87d1-26eef8e2a261" />

- Все работает корректно. Даже если в bookings_old проиндексировать поле book_date     create index date_index on bookings_old(book_date);  то запросы все равно будут выполняться медленней, чем в секционной таблице. В 2 раза дольше \
   explain analyze select * from bookings_old where book_date > '2025-12-01'; \
  <img width="895" height="122" alt="image" src="https://github.com/user-attachments/assets/fb8e5b6d-d943-44ed-8740-aada70316ce4" />

  




