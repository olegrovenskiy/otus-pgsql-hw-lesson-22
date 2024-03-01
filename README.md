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
        otus_hw_22$#
        otus_hw_22$# END;
        otus_hw_22$# $TRIG_FUNC$
        otus_hw_22-#   LANGUAGE plpgsql
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
                         |       |                  |                     |      |            |          |          |          |                   |          |                                                    +|
                         |       |                  |                     |      |            |          |          |          |                   |          | END;                                               +|
                         |       |                  |                     |      |            |          |          |          |                   |          |                                                     |
        (1 row)
        
        otus_hw_22=# ^C
        otus_hw_22=#



CREATE TRIGGER tr_hw_22
AFTER INSERT OR UPDATE OR DELETE
ON sales
FOR EACH STATEMENT
EXECUTE FUNCTION hw_22();





