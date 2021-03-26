
## SQL Practice Exercises

### Tables

**salesperson**

| id  | name    | age | salary |
| --- | :----:  | --- | -----  |
|1    | 'Abe'   | 61  |140000  |
|2    | 'Bob'   | 34  |44000   |
|5    | 'Chris' | 34  |40000   |
|7    | 'Dan'   | 41  |52000   |
|8    | 'Ken'   | 57  |115000  |
|11   | 'Joe'   | 38  |38000   |

**customer**

|id   | name      | city      | industryType|
| --- | :------:  | --------- | :----------:|
|4    | 'Samsonic'| 'pleasant'| 'J'         |
|6    | 'Panasung'| 'oaktown' | 'J'         |
|7    | 'Samony'  | 'Jackson' | 'B'         |
|9    | 'Orange'  | 'Jackson' | 'B'         |

**orders**

|number| order_date | cust_id| salesperson_id| amount|
| --- | :------:    | ------ | :----------:| ------  |
|10   | '1996-08-02'| 4      | 2           | 540     |
|20   | '1999-01-30'| 4      | 8           | 1800    |
|30   | '1995-07-14'| 9      | 1           | 160     |
|40   | '1998-01-29'| 7      | 2           | 2400    |
|50   | '1998-02-03'| 6      | 7           | 600     |
|60   | '1998-03-02'| 6      | 7           | 720     |
|70   | '1998-05-06'| 9      | 7           | 150     |

### Creating the Tables

``` SQL
CREATE TABLE salesperson
(
   id integer NOT NULL,
   name character varying(200) NOT NULL,
   age integer NOT NULL,
   salary integer NOT NULL,
   CONSTRAINT salesperson_pk PRIMARY KEY (id)
);

INSERT INTO salesperson (id, name, age, salary) VALUES
(1, 'Abe', 61, 140000),
(2, 'Bob', 34, 44000),
(5, 'Chris', 34, 40000),
(7, 'Dan', 41, 52000),
(8, 'Ken', 57, 115000),
(11, 'Joe', 38, 38000);

CREATE TABLE customer
(
   id integer NOT NULL,
   name character varying(100) NOT NULL,
   city character varying(100) NOT NULL,
   industryType character varying(100) NOT NULL,
   CONSTRAINT customer_pk PRIMARY KEY (id)
);

INSERT INTO customer (id, name, city, industryType) VALUES
(4, 'Samsonic', 'pleasant', 'J'),
(6, 'Panasung', 'oaktown', 'J'),
(7, 'Samony', 'Jackson', 'B'),
(9, 'Orange', 'Jackson', 'B');


CREATE TABLE orders
(
   number integer NOT NULL,
   cust_id integer NOT NULL,
   salesperson_id integer NOT NULL,
   order_date timestamp NOT NULL,
   amount integer NOT NULL,
   CONSTRAINT order_pk PRIMARY KEY (number),
   CONSTRAINT fk_orders_customer FOREIGN KEY (cust_id) REFERENCES customer(id),
   CONSTRAINT fk_orders_salespersonid FOREIGN KEY (salesperson_id) REFERENCES salesperson(id)
);

INSERT INTO orders (number, order_date, cust_id, salesperson_id, amount) VALUES
(10, '1996-08-02', 4, 2, 540),
(20, '1999-01-30', 4, 8, 1800),
(30, '1995-07-14', 9, 1, 160),
(40, '1998-01-29', 7, 2, 2400),
(50, '1998-02-03', 6, 7, 600),
(60, '1998-03-02', 6, 7, 720),
(70, '1998-05-06', 9, 7, 150);
```

**Given the tables above, find the following:**

1. The names of all salespeople that have an order with Samsonic
2. The names of all salespeople that do not have any order with Samsonic.
3. The names of salespeople that have 2 or more orders.
4. The names and ages of all salespersons must having a salary of 100,000 or greater.
5. What sales people have sold more than 1400 total units?
6. When was the earliest and latest order made to Samony?

___
#### Answers

1. The names of all salespeople that have an order with Samsonic
``` SQL
SELECT sp.name FROM salesperson sp
    INNER JOIN orders as ors ON sp.id = ors.salesperson_id
    INNER JOIN customer as ct ON ors.cust_id = ct.id
    WHERE
        ct.name = 'Samsonic';        
```
2. The names of all salespeople that do not have any order with Samsonic.

``` SQL
WITH order_ as (
    SELECT * FROM orders
        WHERE 
            cust_id = 4 
    )
SELECT DISTINCT sp.name FROM salesperson sp
    LEFT JOIN orders as ors ON sp.id = ors.salesperson_id
        WHERE 
            sp.id NOT IN (SELECT salesperson_id FROM order_);   
```

3. The names of salespeople that have 2 or more orders.

``` SQL
WITH order_ as (
    SELECT salesperson_id, count(*) as amount FROM orders
    GROUP BY salesperson_id
    )
SELECT sp.name FROM salesperson sp
    INNER JOIN order_ as ors ON sp.id = ors.salesperson_id
        WHERE 
            ors.amount >=2;   
```

4. The names and ages of all salespersons must having a salary of 100,000 or greater.

``` SQL
SELECT name, age FROM salesperson
   WHERE salary >= 100000;
```

5. What sales people have sold more than 1400 total units?

``` SQL
WITH order_ as (
        SELECT salesperson_id, SUM(amount) AS total FROM orders
        GROUP BY salesperson_id
        )
SELECT sp.name FROM salesperson sp
    INNER JOIN order_ ors ON ors.salesperson_id = sp.id
       WHERE ors.total > 1400;          
```

6. When was the earliest and latest order made to Samony?

``` SQL
SELECT MIN(order_date), MAX(order_date) FROM orders ors
    INNER JOIN customer ct ON ors.cust_id = ct.id
    WHERE ct.name = 'Samsonic';
```