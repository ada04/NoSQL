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
CONTAINER ID   IMAGE              COMMAND                  CREATED         STATUS         PORTS                                                                     NAMES
b0798e4e53df   cassandra:latest   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp                               compose_node2_1
f793b3ccc2c7   cassandra:latest   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp                               compose_node3_1
3ab8c4be0861   cassandra:latest   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   7000-7001/tcp, 0.0.0.0:9042->9042/tcp, 7199/tcp, 0.0.0.0:9160->9160/tcp   compose_node1_1
```

Смотрим статус:

```bash
docker exec -it compose_node3_1 nodetool status
```

```bash
root@ubuntu2204:~/compose# docker exec -it compose_node1_1 nodetool status
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load        Tokens  Owns (effective)  Host ID                               Rack
UN  172.23.0.3  25.55 KiB   16      68.5%             552029d6-c58b-495d-9776-8fd88ca25837  rack1
UN  172.23.0.4  223.76 KiB  16      73.1%             14456b89-4f86-4f2d-a5eb-13c5689bb3a1  rack1
UN  172.23.0.2  25.55 KiB   16      58.4%             ae4f46d7-0641-48ef-bd4e-0004aae788a8  rack1
```

--docker network create app-tier --driver bridge

### Создадим демонстрационные таблицы

Создаем keyspace **cqlkeyspace**:

```sql
CREATE KEYSPACE cqlkeyspace
   WITH replication = {'class': 'SimpleStrategy', 'replication_factor' : 3};
```

Создадим две таблицы **t** и **t2** в keyspace **cqlkeyspace**:

```sql
USE cqlkeyspace;
CREATE TABLE t (
   id int,
   k int,
   v text,
   PRIMARY KEY (id)
);
CREATE TABLE t2 (
   id int,
   k int,
   v text,
   PRIMARY KEY (id)
);
```

Добавим в них данные

```sql
INSERT INTO t (id, k, v) VALUES (0, 0, 'val0');
INSERT INTO t (id, k, v) VALUES (1, 1, 'val1');

INSERT INTO t2 (id, k, v) VALUES (0, 0, 'val0');
INSERT INTO t2 (id, k, v) VALUES (1, 1, 'val1');
INSERT INTO t2 (id, k, v) VALUES (2, 2, 'val2');
```

Проверим что данные попали в таблицы

```sql
SELECT * FROM t;
SELECT * FROM t2;
```

Результат:

```bash
cqlsh:cqlkeyspace> SELECT * FROM t;

 id | k | v
----+---+------
  1 | 1 | val1
  0 | 0 | val0

(2 rows)
cqlsh:cqlkeyspace> SELECT * FROM t2;

 id | k | v
----+---+------
  1 | 1 | val1
  0 | 0 | val0
  2 | 2 | val2

(3 rows)
```

Создадим второй keyspace **catalogkeyspace**

```sql
CREATE KEYSPACE catalogkeyspace
   WITH replication = {'class': 'SimpleStrategy', 'replication_factor' : 3};
```

Создадим две таблицы **journal** и **magazine** в keyspace **catalogkeyspace**

```sql
USE catalogkeyspace;
CREATE TABLE journal (
   id int,
   name text,
   publisher text,
   PRIMARY KEY (id)
);

CREATE TABLE magazine (
   id int,
   name text,
   publisher text,
   PRIMARY KEY (id)
);
```

Заполним их данными

```sql
INSERT INTO journal (id, name, publisher) VALUES (0, 'Apache Cassandra Magazine', 'Apache Cassandra');
INSERT INTO journal (id, name, publisher) VALUES (1, 'Couchbase Magazine', 'Couchbase');

INSERT INTO magazine (id, name, publisher) VALUES (0, 'Apache Cassandra Magazine', 'Apache Cassandra');
INSERT INTO magazine (id, name, publisher) VALUES (1, 'Couchbase Magazine', 'Couchbase');
```

Проверим что данные попали в таблицы

```sql
SELECT * FROM catalogkeyspace.journal;
SELECT * FROM catalogkeyspace.magazine;
```

Результат:

```bash
cqlsh:catalogkeyspace> SELECT * FROM catalogkeyspace.journal;

 id | name                      | publisher
----+---------------------------+------------------
  1 |        Couchbase Magazine |        Couchbase
  0 | Apache Cassandra Magazine | Apache Cassandra

(2 rows)
cqlsh:catalogkeyspace> SELECT * FROM catalogkeyspace.magazine;

 id | name                      | publisher
----+---------------------------+------------------
  1 |        Couchbase Magazine |        Couchbase
  0 | Apache Cassandra Magazine | Apache Cassandra

(2 rows)
```

Ссылка на лог 01

```sql
```
```bash
```


### Воспользовавшись инструкцией https://cassandra.apache.org/doc/latest/cassandra/operating/backups.html создать бэкап и восстановиться из него.

```bash
```

