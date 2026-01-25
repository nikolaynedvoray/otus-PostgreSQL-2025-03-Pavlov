## Настройка мониторинга
 ### Настройка node_exporter 
 ```
   Добавляем пользователя ``sudo useradd -rs /bin/false node_exporter`` \
   Скачиваем экспортер ``wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz`` \
   Разархивируем и копируем бинарник в /usr/local/bin \
   Создаем юнит system.d  
   nano /etc/systemd/system/node-exporter.service
   
   [Unit]
   Description=Node Exporter  
   Wants=network-online.target  
   After=network-online.target  

   [Service]  
   User=node_exporter  
   Group=node_exporter  
   Type=simple  
   ExecStart=/usr/local/bin/node_exporter  
   Restart=always  
   RestartSec=5  

   [Install]  
   WantedBy=multi-user.target

   systemctl daemon-reload
   systemctl enable node-exporter
   systemctl start node-exporter
   ```
### Настройка postgres_exporter  
  Создаем в postgres служебного пользователя и даем ему доступ
  ```
  CREATE USER postgres_exporter;
  \password postgres_exporter
  ALTER USER postgres_exporter SET SEARCH_PATH TO postgres_exporter,pg_catalog;
  GRANT CONNECT ON DATABASE postgres TO postgres_exporter;
  GRANT pg_monitor to postgres_exporter;
  ```
  Устанавливаем экспортер
  ```
  Создаем группу groupadd --system postgres_exporter
  Создаем пользователя useradd -s /sbin/nologin --system -g postgres_exporter postgres_exporter
  Качаем экспортер wget https://github.com/prometheus-community/postgres_exporter/releases/download/v0.17.1/postgres_exporter-0.17.1.linux-amd64.tar.gz
  tar -zxvf postgres_exporter-0.17.1.linux-amd64.tar.gz
  mv postgres_exporter /usr/local/bin/postgres_exporter
  Меняем владельца бинарника chown postgres_exporter:postgres_exporter /usr/local/bin/postgres_exporter
  Создаем конфигурационный файл DATA_SOURCE_NAME='postgresql://postgres_exporter:postgres@localhost:5432/postgres?sslmode=disable'
  Меняем владельца файла конфигурации chown postgres_exporter:postgres_exporter /etc/postgres_exporter.conf

  Создаем юнит system.d nano /etc/systemd/system/postgres_exporter.service
  [Unit]
  Description=PostgreSQL Exporter
  Wants=network-online.target
  After=network-online.target

  [Service]
  User=postgres_exporter
  Group=postgres_exporter
  Type=simple
  ExecStart=/usr/local/bin/postgres_exporter
  EnvironmentFile=/etc/postgres_exporter.conf

  [Install]
  WantedBy=multi-user.target

  systemctl daemon-reload
  systemctl start postgres_exporter
  Проверяем curl http://localhost:9187/metrics
  ```
### Настройка prometheus
  ```
apt install prometheus
nano /etc/prometheus/prometheus.yml

scrape_configs:
  - job_name: "postgres_exporter"

    static_configs:
      - targets: ["localhost:9187"]

systemctl restart prometheus
```
### Настройка Grafana
Так как репозитории графаны заблокированы в России, то пришлось пойти через контейнеры.
```
docker run -d --name=grafana -p 3000:3000 grafana/grafana
Подключаемся через браузер на http://45.9.26.228:3000/
Настраиваем DataSource http://192.168.1.249:9090
Добавляем дашборд из коллекции с Grafanahttps://grafana.com/grafana/dashboards/455-postgres-overview/
```
Проверяем, что все работет \
<img width="2142" height="1183" alt="image" src="https://github.com/user-attachments/assets/2e75a274-50c3-439f-8874-139235ef0245" />

