- Создаем схему и таблицы из приложенного к ДЗ скрипта. \
  -- ДЗ тема: триггеры, поддержка заполнения витрин

DROP SCHEMA IF EXISTS pract_functions CASCADE; \
CREATE SCHEMA pract_functions;

SET search_path = pract_functions;

-- товары: \
CREATE TABLE goods \
( \
    goods_id    integer PRIMARY KEY, \
    good_name   varchar(63) NOT NULL, \
    good_price  numeric(12, 2) NOT NULL CHECK (good_price > 0.0) \
); \
INSERT INTO goods (goods_id, good_name, good_price) \
VALUES 	(1, 'Спички хозайственные', .50), \
		(2, 'Автомобиль Ferrari FXX K', 185000000.01); 

-- Продажи \
CREATE TABLE sales \
( \
    sales_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY, \
    good_id     integer REFERENCES goods (goods_id), \
    sales_time  timestamp with time zone DEFAULT now(), \
    sales_qty   integer CHECK (sales_qty > 0) \
);

INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);

-- отчет: \
SELECT G.good_name, sum(G.good_price * S.sales_qty) \
FROM goods G \
INNER JOIN sales S ON S.good_id = G.goods_id \
GROUP BY G.good_name; 

-- с увеличением объёма данных отчет стал создаваться медленно
-- Принято решение денормализовать БД, создать таблицу
CREATE TABLE good_sum_mart \
( \
	good_name   varchar(63) NOT NULL, \
	sum_sale	numeric(16, 2)NOT NULL \
);


- Создадим тригерную функцию, записывающую данные в витрину при добавлении, удалении, изменении данных в таблице sales
CREATE OR REPLACE FUNCTION update_good_sum_mart() \
RETURNS TRIGGER \
LANGUAGE plpgsql \
AS $$ \
BEGIN \
    -- Очищаем витрину перед пересчетом
    DELETE FROM good_sum_mart; \
        -- Заполняем витрину актуальными данными
    INSERT INTO good_sum_mart (good_name, sum_sale) \
    SELECT \
        G.good_name, \
        SUM(G.good_price * S.sales_qty) AS sum_sale \
    FROM goods G \
    INNER JOIN sales S ON S.good_id = G.goods_id \
    GROUP BY G.good_name; \
    RETURN NULL; \
END; \
$$;

- Дальше создадим сам тригер AFTER, который будет запускать функцию после вставки, обновления, удаления и очистки таблицы
  CREATE OR REPLACE TRIGGER sales_change_trigger \
AFTER INSERT OR UPDATE OR delete OR TRUNCATE ON sales \
FOR EACH STATEMENT \
EXECUTE FUNCTION update_good_sum_mart();

- Проверим работу тригера.
- Посмотрим, что есть в таблице good_sum_mart   select * from good_sum_mart; \
  <img width="312" height="82" alt="image" src="https://github.com/user-attachments/assets/d2f86647-fb95-4605-8fc9-f7772552ed72" />
- Добвавим строку в таблицу sales \
  insert into sales (good_id, sales_qty) values (1, 11); 
- Проверим изменились ли данные в витрине select * from good_sum_mart;  \
  <img width="308" height="77" alt="image" src="https://github.com/user-attachments/assets/22a70798-d366-428a-951c-e48078c87c58" />

- Поменяем количество проданного товара в одной из строк update sales set sales_qty = '1' where sales_id = '18';
- Проверим изменились ли данные в витрине \
  <img width="307" height="85" alt="image" src="https://github.com/user-attachments/assets/4542d169-191d-45ba-adf1-02d6f9686f10" />
- Удалим продажу автомобиля  delete from sales where good_id = '2';
  
- Проверим витрину \
  
  <img width="297" height="55" alt="image" src="https://github.com/user-attachments/assets/1f44ec33-dc33-46ac-95e2-fa83eeae869e" />

- Теперь очистим таблицу с продажами  truncate table sales; \
- Витрина также очистилась \
  <img width="270" height="72" alt="image" src="https://github.com/user-attachments/assets/66c24a3c-88e4-4aaa-ac53-8783daad2e32" />


  


