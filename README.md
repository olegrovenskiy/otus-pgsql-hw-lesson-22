# ДЗ otus-pgsql-hw-lesson-22


#### 1. Создание БД для ДЗ

    postgres=#
    postgres=# create database  otus_hw_22;
    CREATE DATABASE
    postgres=# \c otus_hw_22;
    You are now connected to database "otus_hw_22" as user "postgres".
    otus_hw_22=#

#### 2. Схема и путь поиска

    otus_hw_22=# DROP SCHEMA IF EXISTS pract_functions CASCADE;
    DROP SCHEMA
    otus_hw_22=# CREATE SCHEMA pract_functions;
    CREATE SCHEMA
    otus_hw_22=#
    otus_hw_22=#  SET search_path = pract_functions, public;
    SET
    otus_hw_22=#

#### 3. Создаём и наполняем таблицы

####### GOODS

        CREATE TABLE goods
        (
            goods_id    integer PRIMARY KEY,
            good_name   varchar(63) NOT NULL,
            good_price  numeric(12, 2) NOT NULL CHECK (good_price > 0.0)
        );

        otus_hw_22=# CREATE TABLE goods
        otus_hw_22-# (
        otus_hw_22(#     goods_id    integer PRIMARY KEY,
        otus_hw_22(#     good_name   varchar(63) NOT NULL,
        otus_hw_22(#     good_price  numeric(12, 2) NOT NULL CHECK (good_price > 0.0)
        otus_hw_22(# );
        CREATE TABLE
        otus_hw_22=#


INSERT INTO goods (goods_id, good_name, good_price) VALUES (1, 'Спички хозайственные', .50), (2, 'Автомобиль Ferrari FXX K', 185000000.01);


        otus_hw_22=#
        otus_hw_22=# INSERT INTO goods (goods_id, good_name, good_price) VALUES (1, 'Спички хозайственные', .50), (2, 'Автомобиль Ferrari FXX K', 185000000.01);
        INSERT 0 2
        otus_hw_22=# select * from goods;
         goods_id |        good_name         |  good_price
        ----------+--------------------------+--------------
                1 | Спички хозайственные     |         0.50
                2 | Автомобиль Ferrari FXX K | 185000000.01
        (2 rows)
        
        otus_hw_22=#

####### SALES

        CREATE TABLE sales
        (
            sales_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
            good_id     integer REFERENCES goods (goods_id),
            sales_time  timestamp with time zone DEFAULT now(),
            sales_qty   integer CHECK (sales_qty > 0)
        );

INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);


    otus_hw_22=#
    otus_hw_22=# CREATE TABLE sales
    otus_hw_22-# (
    otus_hw_22(#     sales_id    integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    otus_hw_22(#     good_id     integer REFERENCES goods (goods_id),
    otus_hw_22(#     sales_time  timestamp with time zone DEFAULT now(),
    otus_hw_22(#     sales_qty   integer CHECK (sales_qty > 0)
    otus_hw_22(# );
    CREATE TABLE
    otus_hw_22=# INSERT INTO sales (good_id, sales_qty) VALUES (1, 10), (1, 1), (1, 120), (2, 1);
    INSERT 0 4
    otus_hw_22=# select * from sales;
     sales_id | good_id |          sales_time           | sales_qty
    ----------+---------+-------------------------------+-----------
            1 |       1 | 2024-03-01 07:05:16.650125-05 |        10
            2 |       1 | 2024-03-01 07:05:16.650125-05 |         1
            3 |       1 | 2024-03-01 07:05:16.650125-05 |       120
            4 |       2 | 2024-03-01 07:05:16.650125-05 |         1
    (4 rows)
    
    otus_hw_22=#

#### 4. Отчёт

        SELECT G.good_name, sum(G.good_price * S.sales_qty)
        FROM goods G
        INNER JOIN sales S ON S.good_id = G.goods_id
        GROUP BY G.good_name;

    otus_hw_22=#
    otus_hw_22=# SELECT G.good_name, sum(G.good_price * S.sales_qty)
    otus_hw_22-# FROM goods G
    otus_hw_22-# INNER JOIN sales S ON S.good_id = G.goods_id
    otus_hw_22-# GROUP BY G.good_name;
            good_name         |     sum
    --------------------------+--------------
     Автомобиль Ferrari FXX K | 185000000.01
     Спички хозайственные     |        65.50
    (2 rows)
    
    otus_hw_22=#


#### 5. Денормализация БД



        otus_hw_22=# CREATE TABLE good_sum_mart
        otus_hw_22-# (
        otus_hw_22(#    good_name  varchar(63) NOT NULL,
        otus_hw_22(#    sum_sale   numeric(16, 2) NOT NULL
        otus_hw_22(# );
        CREATE TABLE
        otus_hw_22=#
        otus_hw_22=#
        otus_hw_22=# \dt
                         List of relations
             Schema      |     Name      | Type  |  Owner
        -----------------+---------------+-------+----------
         pract_functions | good_sum_mart | table | postgres
         pract_functions | goods         | table | postgres
         pract_functions | sales         | table | postgres
        (3 rows)
        
        otus_hw_22=#

#### 6. Создание триггера

Создать триггер на таблице продаж, для поддержки данных в витрине в актуальном состоянии (вычисляющий при каждой продаже сумму и записывающий её в витрину)

1. Создаём триггер функцию


                DELETE FROM good_sum_mart * ;
                INSERT INTO good_sum_mart (good_name, sum_sale)
                SELECT G.good_name, sum(G.good_price * S.sales_qty)
                FROM goods G
                INNER JOIN sales S ON S.good_id = G.goods_id
                GROUP BY G.good_name;
        
                otus_hw_22=#
                otus_hw_22=# INSERT INTO good_sum_mart (good_name, sum_sale)
                otus_hw_22-# SELECT G.good_name, sum(G.good_price * S.sales_qty)
                otus_hw_22-# FROM goods G
                otus_hw_22-# INNER JOIN sales S ON S.good_id = G.goods_id
                otus_hw_22-# GROUP BY G.good_name;
                INSERT 0 2
                otus_hw_22=# select * from good_sum_mart;
                        good_name         |   sum_sale
                --------------------------+--------------
                 Автомобиль Ferrari FXX K | 185000000.01
                 Спички хозайственные     |        65.50
                (2 rows)
                
                otus_hw_22=#
        
                otus_hw_22=# CREATE OR REPLACE FUNCTION hw_22()
                otus_hw_22-# RETURNS trigger
                otus_hw_22-# AS
                otus_hw_22-# $TRIG_FUNC$
                otus_hw_22$# BEGIN
                otus_hw_22$#
                otus_hw_22$# DELETE FROM good_sum_mart * ;
                otus_hw_22$# INSERT INTO good_sum_mart (good_name, sum_sale)
                otus_hw_22$# SELECT G.good_name, sum(G.good_price * S.sales_qty)
                otus_hw_22$# FROM goods G
                otus_hw_22$# INNER JOIN sales S ON S.good_id = G.goods_id
                otus_hw_22$# GROUP BY G.good_name;
                otus_hw_22$# RETURN NULL;
                otus_hw_22$# END;
                otus_hw_22$# $TRIG_FUNC$
                otus_hw_22-#   LANGUAGE plpgsql
                otus_hw_22-#
                otus_hw_22-# ;
                CREATE FUNCTION

                otus_hw_22=#
                otus_hw_22=#
                otus_hw_22=# \df+
                                                                                                            List of functions
             Schema      | Name  | Result data type | Argument data types | Type | Volatility | Parallel |  Owner   | Security | Access privileges | Language |                     Source code                     | Description
        -----------------+-------+------------------+---------------------+------+------------+----------+----------+----------+-------------------+----------+-----------------------------------------------------+-------------
         pract_functions | hw_22 | trigger          |                     | func | volatile   | unsafe   | postgres | invoker  |                   | plpgsql  |                                                    +|
                         |       |                  |                     |      |            |          |          |          |                   |          | BEGIN                                              +|
                         |       |                  |                     |      |            |          |          |          |                   |          |                                                    +|
                         |       |                  |                     |      |            |          |          |          |                   |          | DELETE FROM good_sum_mart * ;                      +|
                         |       |                  |                     |      |            |          |          |          |                   |          | INSERT INTO good_sum_mart (good_name, sum_sale)    +|
                         |       |                  |                     |      |            |          |          |          |                   |          | SELECT G.good_name, sum(G.good_price * S.sales_qty)+|
                         |       |                  |                     |      |            |          |          |          |                   |          | FROM goods G                                       +|
                         |       |                  |                     |      |            |          |          |          |                   |          | INNER JOIN sales S ON S.good_id = G.goods_id       +|
                         |       |                  |                     |      |            |          |          |          |                   |          | GROUP BY G.good_name;                              +|
                         |       |                  |                     |      |            |          |          |          |                   |          | RETURN NULL;                                       +|
                         |       |                  |                     |      |            |          |          |          |                   |          | END;                                               +|
                         |       |                  |                     |      |            |          |          |          |                   |          |                                                     |
        (1 row)
        
        otus_hw_22=#



Создаём триггер

        CREATE TRIGGER tr_hw_22
        AFTER INSERT OR UPDATE OR DELETE
        ON sales
        FOR EACH STATEMENT
        EXECUTE FUNCTION hw_22();


        otus_hw_22=#
        otus_hw_22=# CREATE TRIGGER tr_hw_22
        otus_hw_22-# AFTER INSERT OR UPDATE OR DELETE
        otus_hw_22-# ON sales
        otus_hw_22-# FOR EACH STATEMENT
        otus_hw_22-# EXECUTE FUNCTION hw_22();
        CREATE TRIGGER
        otus_hw_22=#


--------------
#### 7. Проверка работы триггера

        otus_hw_22=#
        otus_hw_22=# select * from good_sum_mart;
                good_name         |   sum_sale
        --------------------------+--------------
         Автомобиль Ferrari FXX K | 185000000.01
         Спички хозайственные     |        75.50
        (2 rows)
        
        otus_hw_22=# select * from sales;
         sales_id | good_id |          sales_time           | sales_qty
        ----------+---------+-------------------------------+-----------
                1 |       1 | 2024-03-01 07:05:16.650125-05 |        10
                2 |       1 | 2024-03-01 07:05:16.650125-05 |         1
                3 |       1 | 2024-03-01 07:05:16.650125-05 |       120
                4 |       2 | 2024-03-01 07:05:16.650125-05 |         1
                5 |       1 | 2024-03-01 07:44:06.199849-05 |        20
        (5 rows)
        
        otus_hw_22=#
        
        
        otus_hw_22=# INSERT INTO sales (good_id, sales_qty) VALUES (1, 25);
        INSERT 0 1
        otus_hw_22=#
        
        otus_hw_22=# select * from sales;
         sales_id | good_id |          sales_time           | sales_qty
        ----------+---------+-------------------------------+-----------
                1 |       1 | 2024-03-01 07:05:16.650125-05 |        10
                2 |       1 | 2024-03-01 07:05:16.650125-05 |         1
                3 |       1 | 2024-03-01 07:05:16.650125-05 |       120
                4 |       2 | 2024-03-01 07:05:16.650125-05 |         1
                5 |       1 | 2024-03-01 07:44:06.199849-05 |        20
                7 |       1 | 2024-03-01 08:52:19.537748-05 |        25
        (6 rows)
        
        otus_hw_22=# select * from good_sum_mart;
                good_name         |   sum_sale
        --------------------------+--------------
         Автомобиль Ferrari FXX K | 185000000.01
         Спички хозайственные     |        88.00
        (2 rows)
        
        otus_hw_22=#
        
        
        otus_hw_22=#
        otus_hw_22=# select * from good_sum_mart;
                good_name         |   sum_sale
        --------------------------+--------------
         Автомобиль Ferrari FXX K | 185000000.01
         Спички хозайственные     |        88.00
        (2 rows)
        
        otus_hw_22=# INSERT INTO sales (good_id, sales_qty) VALUES (1, 30);
        INSERT 0 1
        otus_hw_22=# select * from good_sum_mart;
                good_name         |   sum_sale
        --------------------------+--------------
         Автомобиль Ferrari FXX K | 185000000.01
         Спички хозайственные     |       103.00
        (2 rows)
        
        otus_hw_22=#


Видно что при изменении данных в таблицы sales происходит актуализация таблицы good_sum_mart

#### 8. Чем такая схема (витрина+триггер) предпочтительнее отчета, создаваемого "по требованию" (кроме производительности)?

1. не требуется отслеживать изменения в продажах
2. можно создать дополнительные условия или действия

В задании было - Подсказка: В реальной жизни возможны изменения цен.

Это действительно можно обеспечить с помощью тригера и тригер функции, но тогда схема работы будет следующая:

1. При каждой продаже добавляется дополнителдьная запись в таблицу good_sum_mart;
2. При изменении добавляется дельта, так как у нис в таблице good_sum_mart нет индексов, то будем вставлять изменения
3. При удалении тоже вставляем изменение, а именно старое значение со знаком минус. Для этого изменим триггер с AFTER на BEFORE
4. Таким образом старые продажи будут храниться со старыми ценами, а новые уже будут с новыми ценами

Для тестов создал новую таблицу good_sum_mart_1;

Новый триггер и функция:

        DROP TRIGGER IF EXISTS t_new_01 ON sales;
        CREATE TRIGGER t_new_01
        BEFORE INSERT OR UPDATE OR DELETE
        ON sales
        FOR EACH ROW EXECUTE FUNCTION hw_22_new();
        
        
        CREATE OR REPLACE FUNCTION hw_22_new() 
        RETURNS trigger 
        AS
        $TRIG_FUNC$
        BEGIN
            IF      TG_OP = 'INSERT'
            THEN
                INSERT INTO good_sum_mart_1 (good_name, sum_sale)
                SELECT G.good_name, G.good_price * NEW.sales_qty
                FROM goods G
                INNER JOIN sales S ON S.good_id = G.goods_id
                WHERE S.sales_id=NEW.sales_id;
                    RETURN NEW;
        
        
            ELSIF   TG_OP = 'UPDATE'
            THEN
                INSERT INTO good_sum_mart_1 (good_name, sum_sale)
                SELECT G.good_name, G.good_price * (NEW.sales_qty-OLD.sales_qty)
                FROM goods G
                INNER JOIN sales S ON S.good_id = G.goods_id
                WHERE S.sales_id=NEW.sales_id;
        	      RETURN NEW;
        
        
            ELSIF   TG_OP = 'DELETE'
            THEN
                INSERT INTO good_sum_mart_1 (good_name, sum_sale)
                SELECT G.good_name, G.good_price * (0-OLD.sales_qty)
                FROM goods G
                INNER JOIN sales S ON S.good_id = G.goods_id
                WHERE S.sales_id=OLD.sales_id;
        	      RETURN OLD;
        
                END IF;
        END;
        $TRIG_FUNC$
         LANGUAGE plpgsql;


Протестировал отработку Insert, Update, Delete - соответствует ожиданиям


        otus_hw_22=# INSERT INTO sales (good_id, sales_qty) VALUES (1, 10);
        INSERT 0 1
        otus_hw_22=#
        
        otus_hw_22=# select * from good_sum_mart_1;
              good_name       | sum_sale
        ----------------------+----------
         Спички хозайственные |     4.00
         Спички хозайственные |     4.00
         Спички хозайственные |     5.00
        (3 rows)
        
        
        otus_hw_22=#  update sales set sales_qty = 5 where sales_id = 26;
        UPDATE 1
        otus_hw_22=# select * from good_sum_mart_1;
              good_name       | sum_sale
        ----------------------+----------
         Спички хозайственные |     4.00
         Спички хозайственные |     4.00
         Спички хозайственные |     5.00
         Спички хозайственные |    -2.50
        (4 rows)
        
        
        otus_hw_22=#
        otus_hw_22=#
        otus_hw_22=# delete from sales where sales_id = 23;
        DELETE 1
        otus_hw_22=# select * from good_sum_mart_1;
              good_name       | sum_sale
        ----------------------+----------
         Спички хозайственные |     4.00
         Спички хозайственные |     4.00
         Спички хозайственные |     5.00
         Спички хозайственные |    -2.50
         Спички хозайственные |    -2.00
        (5 rows)
        
        otus_hw_22=#
        


###    Правки после замечаний

1. Создал таблицу витрины, в которой ключ это имя товара и он должен быть уникальный

        CREATE TABLE good_sum_mart_correction 
        (
        good_name  varchar(63) PRIMARY KEY,
        sum_sale   numeric(16, 2) NOT NULL,
        CONSTRAINT good_name_unique UNIQUE (good_name)
        );

2.   Скорректировал триггер и функцию


        DROP TRIGGER IF EXISTS t_correction ON sales;
        CREATE TRIGGER t_correction
        AFTER INSERT OR UPDATE OR DELETE
        ON sales
        FOR EACH ROW EXECUTE FUNCTION hw_22_correction();





        CREATE OR REPLACE FUNCTION hw_22_correction()
        RETURNS trigger
        AS
        $TRIG_FUNC$
        BEGIN
        
        	    UPDATE good_sum_mart_correction
                    SET sum_sale = sum_sale + G.good_price * NEW.sales_qty
                    FROM goods G
                    INNER JOIN sales S ON S.good_id = G.goods_id
                    WHERE good_sum_mart_correction.good_name = G.good_name AND S.sales_id=NEW.sales_id;
        
        	IF NOT FOUND THEN
        
                    INSERT INTO good_sum_mart_correction (good_name, sum_sale)
                    SELECT G.good_name, G.good_price * NEW.sales_qty
                    FROM goods G
                    INNER JOIN sales S ON S.good_id = G.goods_id
                    WHERE S.sales_id=NEW.sales_id;
        	END IF;
         RETURN NULL;
         END;
         $TRIG_FUNC$
         LANGUAGE plpgsql;

3. Проверил, что update действительно отрабатывае

        otus_hw_22=#  select * from good_sum_mart_correction;
              good_name       | sum_sale
        ----------------------+----------
         Спички хозайственные |   137.00
        (1 row)
        
        otus_hw_22=# CREATE OR REPLACE FUNCTION hw_22_correction()
        otus_hw_22-# RETURNS trigger
        otus_hw_22-# AS
        otus_hw_22-# $TRIG_FUNC$
        otus_hw_22$# BEGIN
        otus_hw_22$#
        otus_hw_22$#     UPDATE good_sum_mart_correction
        otus_hw_22$#             SET sum_sale = sum_sale + G.good_price * NEW.sales_qty
        otus_hw_22$#             FROM goods G
        otus_hw_22$#             INNER JOIN sales S ON S.good_id = G.goods_id
        otus_hw_22$#             WHERE good_sum_mart_correction.good_name = G.good_name AND S.sales_id=NEW.sales_id;
        otus_hw_22$#
        otus_hw_22$# IF NOT FOUND THEN
        otus_hw_22$#
        otus_hw_22$#             INSERT INTO good_sum_mart_correction (good_name, sum_sale)
        otus_hw_22$#             SELECT G.good_name, G.good_price * NEW.sales_qty
        otus_hw_22$#             FROM goods G
        otus_hw_22$#             INNER JOIN sales S ON S.good_id = G.goods_id
        otus_hw_22$#             WHERE S.sales_id=NEW.sales_id;
        otus_hw_22$# END IF;
        otus_hw_22$#  RETURN NULL;
        otus_hw_22$#  END;
        otus_hw_22$#  $TRIG_FUNC$
        otus_hw_22-#  LANGUAGE plpgsql;
        CREATE FUNCTION
        otus_hw_22=# INSERT INTO sales (good_id, sales_qty) VALUES (1, 20);
        INSERT 0 1
        otus_hw_22=#  select * from good_sum_mart_correction;
              good_name       | sum_sale
        ----------------------+----------
         Спички хозайственные |   147.00
        (1 row)
        
        otus_hw_22=#


        otus_hw_22=# ^C
        otus_hw_22=# INSERT INTO sales (good_id, sales_qty) VALUES (1, 200);
        INSERT 0 1
        otus_hw_22=#  select * from good_sum_mart_correction;
              good_name       | sum_sale
        ----------------------+----------
         Спички хозайственные |   247.00
        (1 row)
        
        otus_hw_22=#




