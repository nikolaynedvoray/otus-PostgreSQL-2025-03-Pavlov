- Создал базу books. Саму базу и наполнение взял с сайта stepic. Курс "Интерактивный тренажер по SQL". <img width="505" height="417" alt="image" src="https://github.com/user-attachments/assets/06b5f0d3-ba0a-4233-84f4-0b8cab562891" />
- <img width="852" height="1107" alt="image" src="https://github.com/user-attachments/assets/e999c84d-fc3f-49e4-8d9c-0f45fc56a2f4" />
- Соединили таблицы author и book. <img width="428" height="303" alt="image" src="https://github.com/user-attachments/assets/ef3a2058-cc65-4d04-b3b8-0b872a7784bf" /> В выборку не вошел Лермонтов, так как в таблице book нет книг, соответствующих ему.
- Соединил эти же таблицы через LEFT JOIN. <img width="476" height="335" alt="image" src="https://github.com/user-attachments/assets/c793acba-1c0f-4f0d-9355-f96a73fbb36a" /> В выборку вошел Лермонтов, так как левое соединение показывает даже те значения соответствия которым нет. Значение title получилось NULL.
- Соединил таблицы через CROSS JOIN. Получилось декартово произведение всех со всеми. <img width="521" height="927" alt="image" src="https://github.com/user-attachments/assets/8af597e7-6d39-4d63-baff-539ed6783f62" />
- Для наглядности выполнения FULL JOIN добавим в таблицу author author_id = 6 с пустым автором, в таблицу books добавим книгу Гамлет с author_id = 6.
- insert into author (author_id) values (6);   insert into book (title, author_id) values ('Гамлет', 6);
- Теперь выполним FULL JOIN. Запрос вернул все строки не взирая на отсутствие соответствия <img width="506" height="341" alt="image" src="https://github.com/user-attachments/assets/4a591681-91cd-4d26-8b61-ddc794518f20" />
- Соединим author и book через FULL JOIN, а потом соединим с genre через INNER JOIN. В результат не вошел Лермонтов, так как в book нет книг, написанных этим автором. <img width="597" height="323" alt="image" src="https://github.com/user-attachments/assets/bc54e7a9-4880-416d-b049-47818235d40e" />




