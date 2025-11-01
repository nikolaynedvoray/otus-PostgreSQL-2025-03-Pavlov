- Развернул в облаке Cloud.ru платформа Advanced машину с 2VCPU, 4GB RAM, SSD. Установил Postgres.
-  Инициализировал БД в pgbench pgbench -i postgres , запустил pgbench с настройками по умолчанию. Получил tps = 1697.804075 <img width="662" height="523" alt="image" src="https://github.com/user-attachments/assets/3adfacf7-22bf-4be5-ba14-474b791729b0" />
- К сожалению, в пункте "Применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла" не смог найти в материалах прикрепленный файл с настройками. Поэтому сделал настройки из предыдущего задания. После отключения параметра fsync pgbench выдал результат tps = 3782.534580
- Создал таблицу student, забил ее миллионом строк. CREATE TABLE student(
 id serial,
 fio char(100)
);   INSERT INTO student(fio) SELECT 'noname' FROM generate_series(1,500000);
- Размер таблицы 135MB. Сделал 5 раз update. Размер стал 808MB <img width="748" height="426" alt="image" src="https://github.com/user-attachments/assets/be284e1f-ffc2-4132-96d9-adbd35eb5c7a" />
- Посмотрел количество мертвых строк и когда был автовакуум. <img width="872" height="112" alt="image" src="https://github.com/user-attachments/assets/f0fde7b7-b768-44e8-ac5a-ccc5bebd08c4" />
- Проверил через минуту - автовакуум очистил старые версии. <img width="876" height="130" alt="image" src="https://github.com/user-attachments/assets/a31bae24-0055-4c04-8eb5-24558805e353" />
- Отключил автовакуум alter table student set (autovacuum_enabled = false); Сделал 10 раз update. Размер таблицы вырос до 2156MB. Количество мертвых версий выросло до  14998887. Автовакуум отключен, старые версии не чистятся. <img width="877" height="647" alt="image" src="https://github.com/user-attachments/assets/8f6b9896-0a3e-4769-960e-f75bc3bf758a" />




