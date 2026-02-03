## Установка и настройка ETCD кластера.
   Установка: 
```   
  sudo apt -y install etcd-server
  sudo apt -y install etcd-client
  sudo systemctl stop etcd
  sudo systemctl disable etcd
  sudo rm -rf /var/lib/etcd/default --- удалить конфигурацию по умолчанию
```
Настраиваем конфигурационный файл /etc/default/etcd
```
ETCD_NAME="etcd1"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.1.140:2379"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.168.1.140:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd_cluster"
ETCD_INITIAL_CLUSTER="etcd1=http://192.168.1.140:2380,etcd2=http://192.168.1.242:2380,etcd3=http://192.168.1.116:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_ELECTION_TIMEOUT="10000"
ETCD_HEARTBEAT_INTERVAL="2000"
ETCD_INITIAL_ELECTION_TICK_ADVANCE="false"
ETCD_ENABLE_V2="true"
```
Настраиваем автозапуск
```
systemctl daemon-reload
systemctl enable etcd
systemctl start etcd
```
Делаем то же самое на остальных 2 машинах и проверяем etcdctl endpoint status --cluster -w table  
<img width="1528" height="146" alt="image" src="https://github.com/user-attachments/assets/af26c72f-577e-44a8-a0aa-4efc38db8ac3" />

## Настройка Postgres
Настроим pg_hba
```
host replication replicator 127.0.0.1/32 scram-sha-256
host replication replicator 192.168.1.241/32 scram-sha-256
host replication replicator 192.168.1.106/32 scram-sha-256
host all all 0.0.0.0/0 scram-sha-256
```
Настроим postgresql.conf
``listen_address = '*'``  
Создадим пользователя replicator    `create user replicator replication login encrypted password 'postgres';`  
На 2 ноде остановим postgres и удалим каталог с данными.

## Настройка Patroni
На каждой ноде выполним команды
```
 sudo apt -y install python3 python3-pip python3-dev python3-psycopg2 libpq-dev
 sudo pip3 install launchpadlib --break-system-packages
 sudo pip3 install --upgrade setuptools --break-system-packages
 sudo pip3 install psycopg2 --break-system-packages
 sudo pip3 install python-etcd --break-system-packages
 sudo apt -y install patroni
 sudo systemctl stop patroni
 sudo systemctl disable patroni
```
Настроим файл конфигурации /etc/patroni/patroni.yml
```
scope: Patroni-cluster
namespace: /db
name: Node1
restapi:
  listen: 0.0.0.0:8008
  connect_address: 192.168.1.7:8008
etcd:
  hosts: 192.168.1.140:2379,192.168.1.242:2379,192.168.1.116:2379
bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      pg_hba:
      - host replication replicator 192.168.1.241/32 scram-sha-256
      - host replication replicator 192.168.1.7/32 scram-sha-256
      - host all all 0.0.0.0/0 scram-sha-256
      parameters:
        autovacuum_analyze_scale_factor: 0.01
  initdb:
  - encoding: UTF8
postgresql:
  listen: 0.0.0.0:5432
  connect_address: 192.168.1.7:5432
  data_dir: /var/lib/postgresql/18/main
  config_dir: /etc/postgresql/18/main
  bin_dir: /usr/lib/postgresql/18/bin
  authentication:
    replication:
      username: replicator
      password: postgres
    superuser:
      username: postgres
      password: postgres
  parameters:
    unix_socket_directories: '..'  # parent directory of data_dir
tags:
    noloadbalance: false
    clonefrom: false
    nostream: false
```
Создадим юнит systemd
```
[Unit]
Description=High availability PostgreSQL Cluster
After=syslog.target network.target
[Service]
Type=simple:
User=postgres
Group=postgres
ExecStart=/usr/bin/patroni /etc/patroni/patroni.yml
KillMode=process
TimeoutSec=30
Restart=no
[Install]
WantedBy=multi-user.target
```
Настроим автозапуск и запустим сервис
```
systemctl daemon-reload
systemctl enable patroni
systemctl start patroni
systemctl status patroni
```
Проверим работу кластера `patronictl -c /etc/patroni/patroni.yml list`  
<img width="920" height="130" alt="image" src="https://github.com/user-attachments/assets/0b68eb88-4288-47db-b1d0-e01f5d5f1113" />  
Кластер работает.  
Проверим аварийный failover, выключив лидера. Видим, что после аварийной перезагрузки лидер автоматически сменился  
<img width="927" height="132" alt="image" src="https://github.com/user-attachments/assets/077ed92c-5487-4d35-9578-637d299feb3b" />  
Также проверим, работает ли switchower `patronictl -c /etc/patroni/patroni.yml switchover`  
<img width="1035" height="402" alt="image" src="https://github.com/user-attachments/assets/b4f865b4-58ed-483d-8e33-9ffcab880b44" />  





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

