- Развернул VM в облаке Cloud.ru(Sbercloud) платформа Advanced.
- Подключился по SSH.
- Сконфигурировал репозиторий Postgres sudo apt install -y postgresql-common  sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh 
- Установил Postgres sudo apt install postgresql-18
- Переключился на пользователя postgres и подключился к базе  su postgres     psql
- Отключили autocommit  \set AUTOCOMMIT off
- Создаем таблицу, заполняем ее данными
  - create table persons(id serial, first_name text, second_name text);
  - insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
- <img width="1565" height="268" alt="image" src="https://github.com/user-attachments/assets/ed15d085-f117-4fa4-849e-734a231c8969" />
- Посмотрели текущий уровень изоляций транзакций
- <img width="482" height="102" alt="image" src="https://github.com/user-attachments/assets/c4254e60-cbc0-481a-bd15-c713beb4fa6d" />
- Создали еще одну SSH сессию и подключились к базе.
- В первой сессии начали транзакцию. Добавили строку. В првой сессии изменения видны.
- <img width="981" height="253" alt="image" src="https://github.com/user-attachments/assets/c69058f1-d431-43b8-a8da-5319b6007b88" />
- Во второй сессии изменения не видны, так как транзакция не закомичена, и данные не записались.
Сделали в первой сессии COMMIT; Данные во второй сессии отобразились, так как транзакция завершилась, и данные записались, а уровень изоляции READ COMMITTED позволяет видеть в других транзакциях закомиченные данные.
- <img width="390" height="143" alt="image" src="https://github.com/user-attachments/assets/5ecf8de6-2387-4e11-b788-2d2f8dce067c" />
- Начал в обоих сессиях новую транзакцию. Установил уровень изоляции set transaction isolation level repeatable read; Добавил в 1 сессии запись в таблицу. Закоммитил в 1 сессии. В 1 сессии изменения видны. Во 2 сессии изменения не видны, так как Repeatable Read позволяет видеть только те данные, которые были зафиксированы до начала транзакции, но не видны незафиксированные данные и изменения, произведённые другими транзакциями в процессе выполнения данной транзакции.
Сделал COMMIT во 2 сессии. При запросе select * from persons; измененные данные видны так как транзакция завершилась и можно получить доступ к изменениям.
- <img width="431" height="370" alt="image" src="https://github.com/user-attachments/assets/bbec9139-bbfd-4efa-90bb-9f905a4b7bdf" />

