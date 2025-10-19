- Развернул VM в облаке Cloud.ru(Sbercloud) платформа Advanced.
- Подключился по SSH.
- Сконфигурировал репозиторий Postgres sudo apt install -y postgresql-common sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh
- Установил Postgres sudo apt install postgresql-18 . Проверил работу кластера.
- <img width="1077" height="65" alt="image" src="https://github.com/user-attachments/assets/16b5ff6a-b4e6-4aaf-8187-344152d6d277" />
- Переключился на пользователя postgres и подключился к базе su postgres     psql
- Создал таблицу и заполнил ее данными  create table test(c1 text);   insert into test values('1');
- <img width="480" height="188" alt="image" src="https://github.com/user-attachments/assets/6fe38621-5724-4b64-b19d-05cb857b0ffb" />
- Остановил кластер
- <img width="1145" height="83" alt="image" src="https://github.com/user-attachments/assets/83e19c9b-5ce3-4e11-935d-6afd4be4123e" />
- Создал новый диск и подключил его к машине. Диск определился как /dev/vdb. Создал файловую систему и примонтировал к /mnt/data. Сделал владельцем диска юзера postgres   chown -R postgres:postgres /mnt/data/
- Перенес файлы из /var/lib/postgresql в /mnt/data.
- При запуске кластер ругается, так как нет данных.
- <img width="721" height="42" alt="image" src="https://github.com/user-attachments/assets/d30baf00-c04a-4604-8662-9a70a09a96bf" />
- В /etc/posgresql/postgresql.conf установил параметр data_directory = '/mnt/data/postgresql/18/main'. Кластер запустился, так как сменили в конф файле рабочую директорию.
-  <img width="1326" height="61" alt="image" src="https://github.com/user-attachments/assets/46cae886-cdc8-4eee-be22-12d90a3a6b65" />
- Подключился к базе. Все данные на месте.
- <img width="545" height="307" alt="image" src="https://github.com/user-attachments/assets/bba7f2db-df81-4311-b0fd-1c23bc58005f" />
# Со звездочкой.
- Отключил диск и подключил его к другой машине с Postgres. Остановил кластер. Изменил в /etc/postgresql/18/main/postgresql.conf рабочую директорию на /mnt/data/postgresql. Файлы в /var/lib/postgresql удалил.
- Кластер не запускается, так как владелец папки юзер postgres и группа postgres с предыдущей машины. После смены владельца на пользователя postgres chown -R postgres /mnt/data/postgresql/  и группы    chown -R :postgres /mnt/data/postgresql/
-  <img width="1065" height="66" alt="image" src="https://github.com/user-attachments/assets/92083451-ca4f-4044-a2fb-2ec8a4d45a0b" />
- Данные на месте <img width="467" height="117" alt="image" src="https://github.com/user-attachments/assets/b78f1459-8bad-4e85-96c3-e30415c17746" />


