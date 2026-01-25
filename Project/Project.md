## Настройка мониторинга
 - Настройка node_exporter \
    Добавляем пользователя ``sudo useradd -rs /bin/false node_exporter`` \
    Скачиваем экспортер ``wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz`` \
    Разархивируем и копируем бинарник в /usr/local/bin \
    Создаем юнит system.d  
   ```
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
