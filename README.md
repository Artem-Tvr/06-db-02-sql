# Домашнее задание к занятию "6.2. SQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose манифест.

*Решение:*

docker-compose.yml

```
version: "3.1"
services:
  postgres:
    image: postgres:12
    environment:
      POSTGRES_DB: "devops"
      POSTGRES_USER: "devops"
      POSTGRES_PASSWORD: "devops"
      PGDATA: "/var/lib/postgresql/data/pgdata"
    volumes:
      - ../2. Init Database:/docker-entrypoint-initdb.d
      - data-vol:/var/lib/postgresql/data/
      - backup-vol:/backups
    ports:
      - "5432:5432"
volumes:
  data-vol: {}
  backup-vol: {}
```

```
root@cadcixztkv:~/hw-postgresql# psql -h 62.113.102.35 -U devops -d devops
Password for user devops:
psql (12.11 (Ubuntu 12.11-0ubuntu0.20.04.1))
Type "help" for help.

```

```
devops=# \l
                              List of databases
   Name    | Owner  | Encoding |  Collate   |   Ctype    | Access privileges
-----------+--------+----------+------------+------------+-------------------
 devops    | devops | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres  | devops | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | devops | UTF8     | en_US.utf8 | en_US.utf8 | =c/devops        +
           |        |          |            |            | devops=CTc/devops
 template1 | devops | UTF8     | en_US.utf8 | en_US.utf8 | =c/devops        +
           |        |          |            |            | devops=CTc/devops
(4 rows)

```

## Задача 2

В БД из задачи 1:

* создайте пользователя test-admin-user и БД test_db
* в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже)
* предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db
* создайте пользователя test-simple-user
* предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db

Таблица orders:

* id (serial primary key)
* наименование (string)
* цена (integer)

Таблица clients:

* id (serial primary key)
* фамилия (string)
* страна проживания (string, index)
* заказ (foreign key orders)

Приведите:

* итоговый список БД после выполнения пунктов выше,
* описание таблиц (describe)
* SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
* список пользователей с правами над таблицами test_db

*Решение:*

```
devops=# \l
                                  List of databases
   Name    | Owner  | Encoding |  Collate   |   Ctype    |     Access privileges
-----------+--------+----------+------------+------------+----------------------------
 devops    | devops | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres  | devops | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | devops | UTF8     | en_US.utf8 | en_US.utf8 | =c/devops                 +
           |        |          |            |            | devops=CTc/devops
 template1 | devops | UTF8     | en_US.utf8 | en_US.utf8 | =c/devops                 +
           |        |          |            |            | devops=CTc/devops
 test_db   | devops | UTF8     | en_US.utf8 | en_US.utf8 | =Tc/devops                +
           |        |          |            |            | devops=CTc/devops         +
           |        |          |            |            | test_admin_user=CTc/devops

```

```
test_db=# \dt
             List of relations
 Schema |  Name   | Type  |      Owner
--------+---------+-------+-----------------
 public | clients | table | test_admin_user
 public | orders  | table | test_admin_user
(2 rows)

```

```
devops=# GRANT ALL PRIVILEGES ON DATABASE test_db TO test_admin_user;
GRANT

test_db=> GRANT SELECT, UPDATE, INSERT, DELETE ON ALL TABLES IN SCHEMA public TO test_simple_user;
GRANT

test_db=# \du
                                       List of roles
    Role name     |                         Attributes                         | Member of
------------------+------------------------------------------------------------+-----------
 devops           | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
 test_admin_user  |                                                            | {}
 test_simple_user |                                                            | {}

```

```
  grantor     |     grantee      | table_catalog | table_schema | table_name | privilege_type | is_grantable | with_hierarchy
-----------------+------------------+---------------+--------------+------------+----------------+--------------+----------------
 test_admin_user | test_admin_user  | test_db       | public       | orders     | INSERT         | YES          | NO
 test_admin_user | test_admin_user  | test_db       | public       | orders     | SELECT         | YES          | YES
 test_admin_user | test_admin_user  | test_db       | public       | orders     | UPDATE         | YES          | NO
 test_admin_user | test_admin_user  | test_db       | public       | orders     | DELETE         | YES          | NO
 test_admin_user | test_admin_user  | test_db       | public       | orders     | TRUNCATE       | YES          | NO
 test_admin_user | test_admin_user  | test_db       | public       | orders     | REFERENCES     | YES          | NO
 test_admin_user | test_admin_user  | test_db       | public       | orders     | TRIGGER        | YES          | NO
 test_admin_user | test_simple_user | test_db       | public       | orders     | INSERT         | NO           | NO
 test_admin_user | test_simple_user | test_db       | public       | orders     | SELECT         | NO           | YES
 test_admin_user | test_simple_user | test_db       | public       | orders     | UPDATE         | NO           | NO
 test_admin_user | test_simple_user | test_db       | public       | orders     | DELETE         | NO           | NO
 test_admin_user | test_admin_user  | test_db       | public       | clients    | INSERT         | YES          | NO
 test_admin_user | test_admin_user  | test_db       | public       | clients    | SELECT         | YES          | YES
 test_admin_user | test_admin_user  | test_db       | public       | clients    | UPDATE         | YES          | NO
 test_admin_user | test_admin_user  | test_db       | public       | clients    | DELETE         | YES          | NO
 test_admin_user | test_admin_user  | test_db       | public       | clients    | TRUNCATE       | YES          | NO
 test_admin_user | test_admin_user  | test_db       | public       | clients    | REFERENCES     | YES          | NO
 test_admin_user | test_admin_user  | test_db       | public       | clients    | TRIGGER        | YES          | NO
 test_admin_user | test_simple_user | test_db       | public       | clients    | INSERT         | NO           | NO
 test_admin_user | test_simple_user | test_db       | public       | clients    | SELECT         | NO           | YES
 test_admin_user | test_simple_user | test_db       | public       | clients    | UPDATE         | NO           | NO
 test_admin_user | test_simple_user | test_db       | public       | clients    | DELETE         | NO           | NO
(22 rows)

~

```

## Задача 3

Используя SQL синтаксис - наполните таблицы следующими тестовыми данными:

Таблица orders


| Наименование | цена |
| -------------------------- | ---------- |
| Шоколад           | 10       |
| Принтер           | 3000     |
| Книга               | 500      |
| Монитор           | 7000     |
| Гитара             | 4000     |

Таблица clients


| ФИО                                 | Страна проживания |
| ---------------------------------------- | ----------------------------------- |
| Иванов Иван Иванович | USA                               |
| Петров Петр Петрович | Canada                            |
| Иоганн Себастьян Бах | Japan                             |
| Ронни Джеймс Дио         | Russia                            |
| Ritchie Blackmore                      | Russia                            |

Используя SQL синтаксис:

* вычислите количество записей для каждой таблицы
* приведите в ответе:
  * запросы
  * результаты их выполнения.

*Решение:*

```
test_db=> insert into orders VALUES (1, 'Шоколад', 10), (2, 'Принтер', 3000), (3, 'Книга', 500), (4, 'Монитор', 7000), (5, 'Гитара', 4000);
INSERT 0 5
test_db=> insert into clients VALUES (1, 'Иванов Иван Иванович', 'USA'), (2, 'Петров Петр Петрович', 'Canada'), (3, 'Иоганн Себастьян Бах', 'Japan'), (4, 'Ронни Джеймс Дио', 'Russia'), (5, 'Ritchie Blackmore', 'Russia');
INSERT 0 5
test_db=> select count (*) from orders;
 count
-------
     5
(1 row)

test_db=> select count (*) from clients;
 count
-------
     5
(1 row)

test_db=>
```

## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys свяжите записи из таблиц, согласно таблице:


| ФИО                                 | Заказ     |
| ---------------------------------------- | ---------------- |
| Иванов Иван Иванович | Книга     |
| Петров Петр Петрович | Монитор |
| Иоганн Себастьян Бах | Гитара   |

Приведите SQL-запросы для выполнения данных операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.

Подсказк - используйте директиву `UPDATE`.

*Решение:*

```
test_db=> UPDATE clients SET booking = 3 where id = 1;
UPDATE 1
test_db=> UPDATE clients SET booking = 4 where id = 2;
UPDATE 1
test_db=> UPDATE  clients SET booking = 5 where id = 3;
UPDATE 1

```

```
test_db=> SELECT * FROM clients AS c WHERE exists (select id from orders as o where c.booking = o.id);
 id |       lastname       | country | booking
----+----------------------+---------+---------
  1 | Иванов Иван Иванович | USA     |       3
  2 | Петров Петр Петрович | Canada  |       4
  3 | Иоганн Себастьян Бах | Japan   |       5

```

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 (используя директиву EXPLAIN).

Приведите получившийся результат и объясните что значат полученные значения.

*Решение:*

```
test_db=> EXPLAIN SELECT * FROM clients AS c WHERE exists (select id from orders as o where c.booking = o.id);
                               QUERY PLAN
------------------------------------------------------------------------
 Hash Join  (cost=37.00..57.24 rows=810 width=72)
   Hash Cond: (c.booking = o.id)
   ->  Seq Scan on clients c  (cost=0.00..18.10 rows=810 width=72)
   ->  Hash  (cost=22.00..22.00 rows=1200 width=4)
         ->  Seq Scan on orders o  (cost=0.00..22.00 rows=1200 width=4)
(5 rows)

```

Чтение данных из таблицы может выполняться несколькими способами. В нашем случае EXPLAIN сообщает, что используется Seq Scan — последовательное, блок за блоком, чтение данных таблицы. cost - некое сферическое в вакууме понятие, призванное оценить затратность операции. Первое значение 0.00 — затраты на получение первой строки. Второе — 18.10 — затраты на получение всех строк. rows — приблизительное количество возвращаемых строк при выполнении операции Seq Scan. Это значение возвращает планировщик. width — средний размер одной строки в байтах.

Hash. Сначала просматривается (Seq Scan) таблица. Для каждой её строки вычисляется хэш (Hash).

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).

Остановите контейнер с PostgreSQL (но не удаляйте volumes).

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления.

*Решение:*

```
root@cadcixztkv:~/hw-postgresql# docker exec -t a6dae5236fff pg_dump -U devops test_db -f /backups/dump_test.sql
root@cadcixztkv:~/hw-postgresql# docker exec -i b2a9b848e4d7 psql -U devops -d test_db -f /backups/dump_test.sql
```
