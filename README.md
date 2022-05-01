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
create table clients (id serial primary key, "Фамилия" varchar, "Страна проживания" varchar, заказ serial, foreign key (заказ) references  orders(id));
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
grant select, insert, update, delete ON all tables in schema public TO "test-simple-user";
```

- итоговый список БД после выполнения пунктов выше,
- описание таблиц (describe)
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
- список пользователей с правами над таблицами test_db
