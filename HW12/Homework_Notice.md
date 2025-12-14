- Добрый день. Выкладываю дополнение к ДЗ, связанному с замечаниями. Прошу проверить еще раз, так как выявленные замечания не подтвердились.
- Давайте обнулим таблицу sales truncate table sales;
- Тогда select * from sales; 
- <img width="597" height="115" alt="image" src="https://github.com/user-attachments/assets/8d4aef23-bef7-43c8-87df-c0df57b2bae2" />
- Соответственно, витрина опустела также  select * from good_sum_mart;
- <img width="410" height="107" alt="image" src="https://github.com/user-attachments/assets/4ad0e990-4a6b-4320-8d10-732eeaeabd28" />
- Снова заполним sales данными, которые были даны в ДЗ  INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);
- Проверим витрину select * from good_sum_mart;
- <img width="401" height="111" alt="image" src="https://github.com/user-attachments/assets/3f63449e-178a-4674-b11b-0f46806362e6" />
- Добавим строку продажи еще 70 спичек, как вы указали в замечании insert into sales (good_id, sales_qty) values (1, 70); 
- Видим, что строка добавилась в sales select * from sales;
- <img width="653" height="192" alt="image" src="https://github.com/user-attachments/assets/74a88e69-791b-44e1-9cff-fd178c95718f" />
- Смотрим пересчитались ли данные в витрине select * from good_sum_mart;  Данные пересчитались. Продажа 70 спичек учлась
- <img width="373" height="117" alt="image" src="https://github.com/user-attachments/assets/f1e4abd4-1382-43f7-a20b-8d3c6d248172" />


