# Установка Postgres на VM в облаке в контейнере
- Развернули машину в облаке Cloud.ru Advanced. Подключились по SSH.
- Установили Docker через convenience script.
  - curl -fsSL https://get.docker.com -o get-docker.sh
  - sudo sh ./get-docker.sh
- Создаем сеть докер  docker network create pg-net
- Пулим с Docker Hub последнюю версию образа Postgres:  Docker pull postgres:latest
- Запускаем сервер Postgres sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/18/docker postgres   (В 18 версии файлы находятся не в /var/lib/postgresql/data , а в /var/lib/postgresql/18/docker)
<img width="1371" height="60" alt="Docker ps" src="https://github.com/user-attachments/assets/72f1d48f-1372-4a39-ac41-d30f9d5b4c43" />
- Разворачиваем клиент в новом контейнере в той же сети Докер и подключаемся по Hostname к серверу. docker run -it --rm --network pg-net --name pg-client postgres psql -h pg-server -U postgres
- Создали таблицу new_table и пару строк в ней.
    - create table new_table(number integer, data varchar(10));
    - insert into new_table values ('1', 'item1'), ('2', 'item2');
- <img width="827" height="215" alt="image" src="https://github.com/user-attachments/assets/b47ede4b-030a-4608-ab02-7884343156ed" />
- Проверили через netstat, что порт 5432 слушается на 0.0.0.0 и, предварительно открыв порт в Security Group в облаке, подключились удаленно через Dbeaver.
<img width="437" height="332" alt="image" src="https://github.com/user-attachments/assets/c0f12fce-1906-44f8-be5d-8c19b22cffec" />
- Перезапускаю контейнер с сервером. docker stop pg-server   docker start pg-server
- Подключаюсь клиентом. Все данные на месте
- <img width="1028" height="506" alt="image" src="https://github.com/user-attachments/assets/6d6dd52a-9575-41a8-84b7-0b7a63262020" />

