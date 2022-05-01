## Задача 1
Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose манифест.

```
$ cat docker-compose.yml
version: '3.1'
services:

  db:
    image: postgres:12
    restart: always
    environment:
      POSTGRES_PASSWORD: testroot
    volumes:
      - db-data:/var/lib/postgresql/data
      - db-backup:/backup

volumes:
    db-data:
      driver: local
    db-backup:
      driver: local
```

## Задача 2

В БД из задачи 1:

- создайте пользователя test-admin-user и БД test_db
```
create database test_db;
create user "test-admin-user" with encrypted password 'testpass';
```
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже)
```
\c test_db
create table orders (id serial primary key, Наименование varchar, Цена integer);
create table clients (id serial primary key, "Фамилия" varchar, "Страна проживания" varchar, заказ integer, foreign key (заказ) references  orders(id));
create index country on clients("Страна проживания");
```
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db
```
grant all on database test_db to "test-admin-user";
```
- создайте пользователя test-simple-user
```
create user "test-simple-user" with encrypted password "testtest";
```
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db
```
grant select, insert, update, delete ON orders to "test-simple-user";
grant select, insert, update, delete ON clients to "test-simple-user";
```

- итоговый список БД после выполнения пунктов выше,
```
test_db=# \l
                                     List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |       Access privileges        
-----------+----------+----------+------------+------------+--------------------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres                   +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres                   +
           |          |          |            |            | postgres=CTc/postgres
 test_db   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =Tc/postgres                  +
           |          |          |            |            | postgres=CTc/postgres         +
           |          |          |            |            | "test-admin-user"=CTc/postgres

```
- описание таблиц (describe)
```
test_db=# \d orders
                                    Table "public.orders"
    Column    |       Type        | Collation | Nullable |              Default               
--------------+-------------------+-----------+----------+------------------------------------
 id           | integer           |           | not null | nextval('orders_id_seq'::regclass)
 Наименование | character varying |           |          | 
 Цена         | integer           |           |          | 
Indexes:
    "orders_pkey" PRIMARY KEY, btree (id)
Referenced by:
    TABLE "clients" CONSTRAINT "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)

test_db=# \d clients
                                       Table "public.clients"
      Column       |       Type        | Collation | Nullable |               Default               
-------------------+-------------------+-----------+----------+-------------------------------------
 id                | integer           |           | not null | nextval('clients_id_seq'::regclass)
 Фамилия           | character varying |           |          | 
 Страна проживания | character varying |           |          | 
 заказ             | integer           |           |          | 
Indexes:
    "clients_pkey" PRIMARY KEY, btree (id)
Foreign-key constraints:
    "clients_заказ_fkey" FOREIGN KEY ("заказ") REFERENCES orders(id)
```
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
```
SELECT table_catalog, table_name, grantee, privilege_type
FROM information_schema.table_privileges
WHERE table_name = 'orders' or table_name = 'clients';
```
- список пользователей с правами над таблицами test_db
```
table_catalog | table_name |     grantee      | privilege_type 
---------------+------------+------------------+----------------
 test_db       | orders     | postgres         | INSERT
 test_db       | orders     | postgres         | SELECT
 test_db       | orders     | postgres         | UPDATE
 test_db       | orders     | postgres         | DELETE
 test_db       | orders     | postgres         | TRUNCATE
 test_db       | orders     | postgres         | REFERENCES
 test_db       | orders     | postgres         | TRIGGER
 test_db       | orders     | test-simple-user | INSERT
 test_db       | orders     | test-simple-user | SELECT
 test_db       | orders     | test-simple-user | UPDATE
 test_db       | orders     | test-simple-user | DELETE
 test_db       | clients    | postgres         | INSERT
 test_db       | clients    | postgres         | SELECT
 test_db       | clients    | postgres         | UPDATE
 test_db       | clients    | postgres         | DELETE
 test_db       | clients    | postgres         | TRUNCATE
 test_db       | clients    | postgres         | REFERENCES
 test_db       | clients    | postgres         | TRIGGER
 test_db       | clients    | test-simple-user | INSERT
 test_db       | clients    | test-simple-user | SELECT
 test_db       | clients    | test-simple-user | UPDATE
 test_db       | clients    | test-simple-user | DELETE
(22 rows)
```
## Задача 3
Используя SQL синтаксис:

- вычислите количество записей для каждой таблицы
- приведите в ответе:
- - запросы
- - результаты их выполнения.
```
test_db=# Select count(*) from clients;
 count 
-------
     5
(1 row)

test_db=# Select count(*) from orders;
 count 
-------
     5
(1 row)
```
## Задача 4
Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Приведите SQL-запросы для выполнения данных операций.
```
update clients set заказ=(select id from orders where Наименование='Книга') where Фамилия='Иванов Иван Иванович';
update clients set заказ=(select id from orders where Наименование='Монитор') where Фамилия='Петров Петр Петрович';
update clients set заказ=(select id from orders where Наименование='Гитара') where Фамилия='Иоганн Себастьян Бах';
```
Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.
```
test_db=# select * from clients where заказ is not null;
 id |       Фамилия        | Страна проживания | заказ 
----+----------------------+-------------------+-------
  1 | Иванов Иван Иванович | USA               |     3
  2 | Петров Петр Петрович | Canada            |     4
  3 | Иоганн Себастьян Бах | Japan             |     5
(3 rows)
```

## Задача 5
```
                                              QUERY PLAN                                              
-----------------------------------------------------------------------------------------------------
 Seq Scan on clients  (cost=0.00..18.10 rows=806 width=72) (actual time=0.008..0.009 rows=3 loops=1)
   Filter: ("заказ" IS NOT NULL)
   Rows Removed by Filter: 2
 Planning Time: 0.033 ms
 Execution Time: 0.019 ms
(5 rows)
```
План запроса : последовательное сканирование таблицы clients с фильтром. Указана стоимость запроса (секунд до начала вывода данных), ожидаемое количество строк, ожидаемая длинна строк. Так же указано сколько реально стоил запрос, сколько строк выведено, сколько циклов потребовалось. Указан используемый фильтр и сколько строк им отфильтровано.  Далее указано время планирования запроса, и время его работы.

## Задача 6
Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).
```
pg_dumpall -U postgres >/backup/backup.dmp
```
Остановите контейнер с PostgreSQL (но не удаляйте volumes).
```
docker stop postgres12
```
Поднимите новый пустой контейнер с PostgreSQL.
```
docker volume rm postgres_db-data
docker-compose up
```
Восстановите БД test_db в новом контейнере.
```
docker exec -ti postgres12 /bin/bash
# psql -U postgres /backup/backup.dmp
```
