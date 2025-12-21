- Создадим таблицы test и test2  CREATE TABLE test(i int); CREATE TABLE test2(i int);
- Изменим уровень логирования WAL на logical ALTER SYSTEM SET wal_level = logical; 
- Перезагрузим кластер и проверим, что изменения применились
  <img width="1132" height="120" alt="image" src="https://github.com/user-attachments/assets/df09f6a2-48cf-4141-a674-379b8466c533" />
- Создадим публикацию CREATE PUBLICATION test_pub FOR TABLE test;
- Проверим, что публикация создалась \dRp+
  <img width="808" height="142" alt="image" src="https://github.com/user-attachments/assets/7329dbb7-0f2c-4cbd-992e-9c84b9ec025d" />
- Далее идем на 2 ВМ, создаем там также базу  test и таблицы test и test2. Также выставляем уровень wal = logical
- Создаем публикацию на таблицу test2
  <img width="760" height="182" alt="image" src="https://github.com/user-attachments/assets/433b7116-3256-4acc-94a8-4e7f602fc6d1" />
- Идем на ВМ2 и создаем подписку

    <img width="637" height="232" alt="image" src="https://github.com/user-attachments/assets/78515a96-4df4-4a72-9b6e-7fc425640bd1" />
- Теперь создадим на ВМ1 подписку на публикацию таблицы table2 с ВМ2
  <img width="612" height="222" alt="image" src="https://github.com/user-attachments/assets/319d4541-73ba-412b-a651-27dcab2844d6" />
- На ВМ3 сделаем те же таблицы. Подпишем таблицу test1 на test1 в ВМ1 и таблицу test2 на test2 в ВМ2
  <img width="688" height="353" alt="image" src="https://github.com/user-attachments/assets/4b75f4d8-3f77-472b-a16d-d1b9bb29f83b" />
- Теперь проверим, что все работает. Внесем на ВМ1 данные в таблицу test - видим, что они появились на ВМ2 и ВМ2
 <img width="1042" height="180" alt="image" src="https://github.com/user-attachments/assets/76fa847e-a701-4f29-a6f8-c6ead58c235a" />
- На ВМ2 внесем данные в test2 - видим, что данные появились на ВМ1 и ВМ3
  <img width="1067" height="247" alt="image" src="https://github.com/user-attachments/assets/6ad59ed1-3244-4c5f-b005-dce15d8cb8c8" />

- Теперь давайте настроим физическую репликацию
- Клонируем из ВМ3 новую машину ВМ4. Остановим кластер. Удалим каталог с данными  /var/lib/postgresql/16/main/
- На ВМ3 в pg_hba.conf настроим replication privilege. выставим  host    replication     all             0.0.0.0/0            scram-sha-256
- Перезагрузим кластер. Забекапим данные с ключом R для репликации pg_basebackup -h 192.168.1.14 --pgdata=/root/backup -R -U postgres
- В backup появился файл postgresql.auto.conf с параметрами для подключения
- Копируем каталог на ВМ4 scp -r /root/backup/ root@192.168.1.97:/var/lib/postgresql/16/main
- Так как копировали под рутом назначаем владельца каталога на ВМ4 пользователя postgres chown -R postgres: /var/lib/postgresql/16/main/
- Стартуем кластер на ВМ4
- <img width="949" height="85" alt="image" src="https://github.com/user-attachments/assets/3da19761-0d1f-442d-bcef-6b6817d44df3" />
- Проверим, как все работаем. На ВМ3 добавим пару записей. Видим, что на ВМ4 они появились
  <img width="886" height="252" alt="image" src="https://github.com/user-attachments/assets/f250014c-8752-46a1-bfc0-c9b644a610fd" />
- Создадим новую таблицу. Видим, что она среплицировалась
- <img width="906" height="195" alt="image" src="https://github.com/user-attachments/assets/d0c054a9-05d0-4f2e-9f80-4cd6cce68a13" />
- При попытке на ВМ4 что-то удалить получим ошибку
  <img width="587" height="51" alt="image" src="https://github.com/user-attachments/assets/7b46ee9a-7b1a-432d-bdce-0ed8010098d1" />
- Судя по статистике репликация идет
  
  <img width="573" height="452" alt="image" src="https://github.com/user-attachments/assets/88d5dffd-c58f-4718-8b21-6a71bb281129" />










