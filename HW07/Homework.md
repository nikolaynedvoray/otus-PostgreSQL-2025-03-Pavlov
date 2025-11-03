- Поменял интервал чекпоинтов до 30 секунд alter system set checkpoint_timeout = '30'; <img width="867" height="127" alt="image" src="https://github.com/user-attachments/assets/9ac49164-47ef-4917-856b-0909dc80570c" />
- Получили текущий LSN  6/7B8C250. Запустили pgbench на 10 минут. После окончания снова получили текущий LSN 6/35CB0C60. Вычислили разницу. За время выполнения pgbench журнал увеличился на 737MB.В среднем по 36MB на одну контрольную точку. <img width="867" height="830" alt="image" src="https://github.com/user-attachments/assets/9e86fabe-8b05-419b-a36e-b8c1d154fb17" />
- За время выполнения pgbench создалось 20 чекпоинтов каждые 30 секунд <img width="1062" height="477" alt="image" src="https://github.com/user-attachments/assets/7fe9fa5c-2bd6-4fc6-a999-93c08755254c" />
- Вернул параметр fsync = on. Запустил pgbench. Получил tps = 1030.085515. Отключили синхронные коммиты alter system set synchronous_commit = 'off'. Запустил pgbench еще раз. Получил tps = 1812.895520. Прирост за счет уменьшения времени подтверждения записи транзакции на диск. <img width="657" height="635" alt="image" src="https://github.com/user-attachments/assets/8f2f4569-3246-48e2-b453-cc7eaa727fff" />
- Создали новый кластер с проверкой контрольной суммы страниц pg_createcluster --datadir=/var/lib/postgresql/18/new_cluster --start 18 new_cluster -- --data-checksums <img width="1587" height="560" alt="image" src="https://github.com/user-attachments/assets/0a9c662a-85e1-4c9f-9ea8-ce32d4f8e3f0" />
- Создали простую таблицу new и вставили в нее пару строк.
- Запросом определили, где она находится на диске и добавили пару бит. <img width="411" height="127" alt="image" src="https://github.com/user-attachments/assets/c64aa74a-7b52-42e1-886a-5fcc70ad3b80" />
- Запустили кластер. При попытке прочитать из таблицы получили ошибку <img width="713" height="47" alt="image" src="https://github.com/user-attachments/assets/55459173-0d65-4f5d-816a-740c3f4a3780" />
- К сожалению применение параметра SET ignore_checksum_failure = on; не помогло. Все равно ошибка повреждения.




