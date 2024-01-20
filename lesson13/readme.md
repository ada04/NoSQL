# Урок 13 - Применение Cassandra на практике. Создание проекта. Часть 2

Масштабирование и отказоустойчивость Cassandra. Часть 1

# Домашнее задание

Масштабирование и отказоустойчивость Cassandra. Часть 2

## Цель:
В результате выполнения ДЗ вы изучите возможности восстановления Cassandra кластеров.

- Описание/Пошаговая инструкция выполнения домашнего задания:
- Воспользовавшись инструкцией https://cassandra.apache.org/doc/latest/cassandra/operating/backups.html создать бэкап и восстановиться из него.
- Какие вы видите подводные камни в этом процессе?
Задание с **:
- воспользоваться сторонними средствами для бэкапа всего кластера, например 3dnap:
- https://portworx.com/blog/kubernetes-cassandra-run-ha-cassandra-rancher-kubernetes-engine/
- примерный конфиг для 3DSnap https://github.com/aeuge/noSqlOtus/tree/main/cassandra_second

## Описание/Пошаговая инструкция выполнения домашнего задания:

### Разворачиваем docker локально

### поднимем 3 узловый Cassandra кластер. 

**docker-compose**

Создадим файл конфигурации кластера:

```bash
#apt install docker-compose
mkditop+ compose
cd compose
vi docker-compose.yml
```

Файл docker-compose.yml:

```bash
version: '2'
services:

  node1:
    image: cassandra:latest
    ports:
      - "9042:9042"
      - "9160:9160"
    environment:
      CASSANDRA_CLUSTER_NAME: demo
      CASSANDRA_SEEDS: node1,node2,node3
    restart: unless-stopped

  node2:
    image: cassandra:latest
    environment:
      CASSANDRA_CLUSTER_NAME: demo
      CASSANDRA_SEEDS: node1,node2,node3
    restart: unless-stopped

  node3:
    image: cassandra:latest
    environment:
      CASSANDRA_CLUSTER_NAME: demo
      CASSANDRA_SEEDS: node1,node2,node3
    restart: unless-stopped
```

Запускаем кластер

```bash
docker-compose up -d
```

```bash
root@ubuntu2204:~/compose# docker-compose up -d
Creating network "compose_default" with the default driver
Creating compose_node3_1 ... done
Creating compose_node1_1 ... done
Creating compose_node2_1 ... done
root@ubuntu2204:~/compose# docker ps
CONTAINER ID   IMAGE              COMMAND                  CREATED          STATUS          PORTS                                                                     NAMES
04a3643c6f70   cassandra:latest   "docker-entrypoint.s…"   18 seconds ago   Up 15 seconds   7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp                               compose_node3_1
552a55602112   cassandra:latest   "docker-entrypoint.s…"   18 seconds ago   Up 15 seconds   7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp                               compose_node2_1
7286b9260aa2   cassandra:latest   "docker-entrypoint.s…"   18 seconds ago   Up 15 seconds   7000-7001/tcp, 0.0.0.0:9042->9042/tcp, 7199/tcp, 0.0.0.0:9160->9160/tcp   compose_node1_1
```

Смотрим статус:

```bash
docker exec -it compose_node3_1 nodetool status
```

```bash
root@ubuntu2204:~/compose# docker exec -it compose_node3_1 nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UN  172.22.0.2  70.28 KiB  16      71.4%             670d2fe9-da30-471f-b8a4-d9e4d77128e1  rack1
UN  172.22.0.3  70.27 KiB  16      69.2%             3a7255af-a69a-43c9-b6de-b8ab744614b6  rack1
UN  172.22.0.4  70.32 KiB  16      59.4%             6c18fb9e-ce59-4fde-b0b6-7b6a4e3dd684  rack1
```

--docker network create app-tier --driver bridge

### Создать keyspaсe с 2-мя таблицами. Одна из таблиц должна иметь составной Partition key, как минимум одно поле - clustering key, как минимум одно поле не входящее в primiry key.

Создаём пространство

```sql
CREATE KEYSPACE IF NOT EXISTS sales WITH replication = {'class': 'SimpleStrategy', 'replication_factor' : 3};
```

```bash
root@ubuntu2204:~# docker exec -it compose_node1_1 cqlsh
Connected to demo at 127.0.0.1:9042
[cqlsh 6.1.0 | Cassandra 4.1.3 | CQL spec 3.4.6 | Native protocol v5]
Use HELP for help.
cqlsh> CREATE KEYSPACE IF NOT EXISTS sales WITH REPLICATION =
   ... { 'class': 'SimpleStrategy',
   ... 'replication_factor': '3'
   ... };
cqlsh>
```

Создаем две таблицы

```bash
CREATE TABLE sales.customers (
       CustomerID varchar,
       CompanyName varchar,
       ContactName varchar,
       ContactTitle varchar,
       Address varchar,
       City varchar,
       Region varchar,
       PostalCode varchar,
       Country varchar,
       Phone varchar,
       Fax varchar,
       PRIMARY KEY (CustomerID)
);

CREATE TABLE sales.orders (
  order_id int,
  customerid text,
  ordertime timestamp,
  items map<int,int>,
  PRIMARY KEY (customerid, ordertime)
);
```

### Заполнить данными обе таблицы.

Заполняем данными (однй из csv файла, вторую при помощи INSERT)

```sql
COPY sales.customers (CustomerID, CompanyName, ContactName, ContactTitle, Address, City, Region, PostalCode, Country, Phone, Fax) FROM '/data/files/customers.csv' WITH HEADER = TRUE;

INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (101000,'CENTC','2023-06-01 12:06:00',{27:1,48:1,116:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100967,'TORTU','2023-05-31 12:05:00',{147:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100968,'PERIC','2023-05-31 12:05:00',{147:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100969,'ERNSH','2023-05-31 12:05:00',{21:1,26:1,30:1,116:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100970,'FRANS','2023-05-31 12:05:00',{21:1,26:1,30:1,40:1,46:1,128:1,147:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100971,'REGGC','2023-05-31 12:05:00',{26:1,35:1,46:1,116:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100972,'QUEDE','2023-05-31 12:05:00',{35:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100973,'MEREP','2023-05-31 12:05:00',{46:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100974,'COMMI','2023-05-31 12:05:00',{30:1,35:1,46:1,48:1,147:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100975,'CENTC','2023-05-31 12:05:00',{21:1,26:1,128:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100976,'TORTU','2023-06-01 12:06:00',{15:1,26:1,35:1,116:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100977,'PERIC','2023-06-01 12:06:00',{21:1,30:1,35:1,40:1,48:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100978,'ERNSH','2023-06-01 12:06:00',{18:1,35:1,147:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100979,'FRANS','2023-06-01 12:06:00',{147:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100980,'REGGC','2023-06-01 12:06:00',{26:1,48:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100981,'QUEDE','2023-06-01 12:06:00',{30:1,46:1,147:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100982,'MEREP','2023-06-01 12:06:00',{21:1,26:1,46:1,48:1,128:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100984,'COMMI','2023-06-01 12:06:00',{30:1,46:1,147:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100985,'CENTC','2023-06-01 12:06:00',{21:1,26:1,48:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100986,'TORTU','2023-06-01 12:06:00',{21:1,26:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100988,'PERIC','2023-06-01 12:06:00',{21:1,30:1,35:1,116:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100989,'ERNSH','2023-06-01 12:06:00',{21:1,26:1,35:1,46:1,48:1,116:1,147:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100990,'FRANS','2023-06-01 12:06:00',{26:1,30:1,46:1,147:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100991,'REGGC','2023-06-01 12:06:00',{26:1,46:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100992,'QUEDE','2023-06-01 12:06:00',{21:1,30:1,46:1,147:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100993,'MEREP','2023-06-01 12:06:00',{26:1,27:1,40:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100994,'COMMI','2023-06-01 12:06:00',{26:1,30:1,128:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100995,'CENTC','2023-06-01 12:06:00',{21:1,30:1,46:1,116:1,147:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100996,'TORTU','2023-06-01 12:06:00',{30:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100997,'PERIC','2023-06-01 12:06:00',{21:1,30:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100998,'ERNSH','2023-06-01 12:06:00',{40:1});
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100999,'FRANS','2023-06-01 12:06:00',{26:1,35:1,116:1});
```

```sql
cqlsh> SELECT count(*) FROM sales.customers;

 count
-------
    91

(1 rows)

Warnings :
Aggregation query used without partition key

cqlsh> SELECT count(*) FROM sales.orders;

 count
-------
    18

(1 rows)

Warnings :
Aggregation query used without partition key

cqlsh>
```

### Воспользовавшись инструкцией https://cassandra.apache.org/doc/latest/cassandra/operating/backups.html создать бэкап и восстановиться из него.

```bash
```

