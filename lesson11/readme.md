# Урок 11 - Cassandra

Масштабирование и отказоустойчивость Cassandra. Часть 1

# Домашнее задание

## Цель:
В результате выполнения ДЗ вы подготовите среду и развернете Cassandra кластер для дальнейшего изучения возможностей маштабирования и восстанавления Cassandra кластеров.

- развернуть docker локально или в облаке
- поднять 3 узловый Cassandra кластер.
- Создать keyspase с 2-мя таблицами. Одна из таблиц должна иметь составной Partition key, как минимум одно поле - clustering key, как минимум одно поле не входящее в primiry key.
- Заполнить данными обе таблицы.
- Выполнить 2-3 варианта запроса использую WHERE
- Создать вторичный индекс на поле, не входящее в primiry key.
- (*) нагрузить кластер при помощи Cassandra Stress Tool (используя "How to use Apache Cassandra Stress Tool.pdf" из материалов).

## Описание/Пошаговая инструкция выполнения домашнего задания:

### Разворачиваем docker локально

```bash
apt install docker.io
```

Проверяем...

```bash
docker run hello-world
```

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

Docker успешно установлен

### поднять 3 узловый Cassandra кластер. (Вариант 1)

#### Скачиваем образ и запускаем первую ноду
    
```bash
docker pull cassandra
docker run --name node1 -d cassandra
```

```bash
root@ubuntu2204:~# docker run --name node1 -d cassandra
3c7cfa133b2bba90e97793887845f1a6e068feb3a881c4bd64f3110dafa512f6
root@ubuntu2204:~# docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS         PORTS                                         NAMES
3c7cfa133b2b   cassandra   "docker-entrypoint.s…"   6 seconds ago   Up 3 seconds   7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp   node1
```

Подключимся по ssh к созданной ноде:
- **docker exec** - выполняет команду в контейнере
- опция **-it** перенаправляет вывод в текущую консоль
- **nodetool status** информация о контейнере
- **cqlsh** CQL shell

```bash
docker exec -it node1 bash
docker exec -it node1 nodetool status
docker exec -it node1 cqlsh
```

```bash
root@ubuntu2204:~# docker exec -it node1 bash
root@3c7cfa133b2b:/# exit
exit
root@ubuntu2204:~# docker exec -it node1 nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load        Tokens  Owns (effective)  Host ID                               Rack
UN  172.17.0.2  104.33 KiB  16      100.0%            09796465-fdcc-42f5-8eef-d5e20db963dc  rack1
```

#### Добавляем еще две ноды 

```bash
docker run --name node2 -d --link node1:cassandra cassandra
docker run --name node3 -d --link node1:cassandra cassandra
```

```bash
root@ubuntu2204:~# docker run --name node2 -d --link node1:cassandra cassandra
02a8e8c9f9741b1250c4dfd160e7bae580d13b0e3fa8337ec8f2713184c514a9
root@ubuntu2204:~# docker run --name node3 -d --link node1:cassandra cassandra
28de717688347d748a83f769a68ad9863a06430fa7b06a399fde5d86614a60fd
```

Посмотрим еще раз информацию о кластере и список запущенных контейнеров.

```bash
docker exec -it node1 nodetool status
```

```bash
root@ubuntu2204:~# docker exec -it node1 nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load        Tokens  Owns (effective)  Host ID                               Rack
UN  172.17.0.4  131.35 KiB  16      100.0%            3c5463a7-d366-4ebc-949c-a4100ffcae35  rack1
UN  172.17.0.2  169.82 KiB  16      100.0%            09796465-fdcc-42f5-8eef-d5e20db963dc  rack1
```

    docker ps -a
    
```bash
root@ubuntu2204:~# docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS         PORTS                                         NAMES
28de71768834   cassandra   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp   node3
02a8e8c9f974   cassandra   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp   node2
3c7cfa133b2b   cassandra   "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp   node1
```

Следующие команды позволяют остановить контейнер и, если он нам более не понадобится, то удалить.

    docker stop node1 node2 node3
    docker rm node1 node2 node3

### поднять 3 узловый Cassandra кластер. (Вариант 2) 

**В этом варианте используем автоматизацию с docker-compose**

Создадим файл конфигурации кластера:

```bash
#apt install docker-compose
mkdir compose
cd compose
vi docker-compose.yml
```

Файл docker-compose.yml:

```bash
version: '2'
services:

  node11:
    image: cassandra:latest
    ports:
      - "9042:9042"
      - "9160:9160"
    environment:
      CASSANDRA_CLUSTER_NAME: demo
      CASSANDRA_SEEDS: node11,node12,node13
    restart: unless-stopped

  node12:
    image: cassandra:latest
    environment:
      CASSANDRA_CLUSTER_NAME: demo
      CASSANDRA_SEEDS: node11,node12,node13
    restart: unless-stopped

  node13:
    image: cassandra:latest
    environment:
      CASSANDRA_CLUSTER_NAME: demo
      CASSANDRA_SEEDS: node11,node12,node13
    restart: unless-stopped
```

Запускаем кластер

```bash
docker-compose up -d
```

```bash
root@ubuntu2204:~/compose# docker-compose up -d
Creating network "compose_default" with the default driver
Creating compose_node13_1 ... done
Creating compose_node11_1 ... done
Creating compose_node12_1 ... done
root@ubuntu2204:~/compose# docker ps
CONTAINER ID   IMAGE              COMMAND                  CREATED          STATUS          PORTS                                                                                                                      NAMES
a32e45a2c3a0   cassandra:latest   "docker-entrypoint.s…"   19 seconds ago   Up 12 seconds   7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp                                                                                compose_node13_1
18c48f67eff3   cassandra:latest   "docker-entrypoint.s…"   19 seconds ago   Up 12 seconds   7000-7001/tcp, 0.0.0.0:9042->9042/tcp, 7199/tcp, 0.0.0.0:9                                                 160->9160/tcp   compose_node11_1
d09ef7caf931   cassandra:latest   "docker-entrypoint.s…"   19 seconds ago   Up 12 seconds   7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp                                                                                compose_node12_1
```

Смотрим статус:

```bash
docker exec -it compose_node11_1 nodetool status
```

```bash
root@ubuntu2204:~/compose# docker exec -it compose_node11_1 nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load        Tokens  Owns (effective)  Host ID                               Rack
UN  172.18.0.4  242.39 KiB  16      70.4%             4a5ae988-ac02-4db5-abd3-a9e5e5aa9def  rack1
UN  172.18.0.3  70.27 KiB   16      58.2%             b841b9a1-d5f2-48ee-bc0d-d4f1c948876b  rack1
UN  172.18.0.2  70.27 KiB   16      71.5%             0ba945de-0e6d-4cbe-92d7-bf3559922bed  rack1
```

--docker network create app-tier --driver bridge

### Создать keyspaсe с 2-мя таблицами. Одна из таблиц должна иметь составной Partition key, как минимум одно поле - clustering key, как минимум одно поле не входящее в primiry key.

Создаём пространство

```sql
CREATE KEYSPACE IF NOT EXISTS sales WITH replication = {'class': 'SimpleStrategy', 'replication_factor' : 3};
};
```

```bash
root@ubuntu2204:~# docker exec -it compose_node11_1 cqlsh
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
INSERT INTO sales.orders (order_id, customerid, ordertime, items) VALUES (100967,'TORTU','2023-05-31 31.05.2000  12:05:00',{147:1});
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
    17

(1 rows)

Warnings :
Aggregation query used without partition key

cqlsh>
```

### Выполнить 2-3 варианта запроса используя WHERE

```sql
cqlsh> SELECT items FROM sales.orders WHERE customerid='COMMI';

 items
--------------------------------------
 {30: 1, 35: 1, 46: 1, 48: 1, 147: 1}
               {26: 1, 30: 1, 128: 1}

(2 rows)
cqlsh> select count(*) from sales.customers where Country = 'France';
InvalidRequest: Error from server: code=2200 [Invalid query] message="Cannot execute this query as it might involve data filtering and thus may have unpredictable performance. If you want to execute this query despite the performance unpredictability, use ALLOW FILTERING"
```


### Создать вторичный индекс на поле, не входящее в primary key.

```sql
CREATE INDEX customers_country_idx ON sales.customers (country);

cqlsh> SELECT count(*) FROM sales.customers WHERE Country = 'France';

 count
-------
    11

(1 rows)

Warnings :
Aggregation query used without partition key
```


### (*) нагрузить кластер при помощи Cassandra Stress Tool (используя "How to use Apache Cassandra Stress Tool.pdf" из материалов).

Ни в одном из двух уроков нет данного документа.
Да и ссылки в уроке ведут на сайт, заблокированный РКН (canva)...
