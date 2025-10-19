# Установка Postgres на VM в облаке в контейнере
- Развернули машину в облаке Cloud.ru Advanced. Подключились по SSH.
- Установили Docker через convenience script.
  - curl -fsSL https://get.docker.com -o get-docker.sh
  - sudo sh ./get-docker.sh
- Пулим с Docker Hub последнюю версию образа Postgres:  Docker pull postgres:latest
- Запускаем сервер Postgres sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/18/docker postgres   (В 18 версии файлы находятся не в /var/lib/postgresql/data , а в /var/lib/postgresql/18/docker)
