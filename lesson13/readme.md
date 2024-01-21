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

Ссылка на лог [Log](./log01.txt))

### Snapshots

Создадим снимки используя команду **nodetool snapshot**

```bash
docker exec -it compose_node1_1 nodetool snapshot
```

```bash
root@ubuntu2204:~/compose# docker exec -it compose_node1_1 nodetool snapshot
Requested creating snapshot(s) for [all keyspaces] with snapshot name [1705836798898] and options {skipFlush=false}
Snapshot directory: 1705836798898
```

#### Настройка снепшотов:

**отвлечение от темы - установим VIm в контейнер**

```bash
docker exec -it compose_node1_1 bash
apt-get update
apt-get install vim
```

Изменим параметры в **cassandra.yaml**

```bash
docker exec -it compose_node1_1 vi /etc/cassandra/cassandra.yaml
```

Выключим автоматическое создание снимков
```bash
auto_snapshot: false
snapshot_before_compaction: false
```

#### Созадим снепшот

Перед созданием снимка посмотрим какие снимки уже есть:

**далее комнады выполняются внутри контейнера**

```bash
docker exec -it compose_node1_1 bash
```

```bash
find -name snapshots
```

```bash
./var/lib/cassandra/data/cqlkeyspace/t2-ea230eb0b84e11ee85a6ddcd8bd8c5a1/snapshots
./var/lib/cassandra/data/cqlkeyspace/t-e797cff0b84e11ee85a6ddcd8bd8c5a1/snapshots
./var/lib/cassandra/data/catalogkeyspace/journal-14c906b0b84f11ee85a6ddcd8bd8c5a1/snapshots
./var/lib/cassandra/data/catalogkeyspace/magazine-196a0610b84f11ee85a6ddcd8bd8c5a1/snapshots
./var/lib/cassandra/data/system_schema/indexes-0feb57ac311f382fba6d9024d305702f/snapshots
./var/lib/cassandra/data/system_schema/functions-96489b7980be3e14a70166a0b9159450/snapshots
./var/lib/cassandra/data/system_schema/types-5a8b1ca866023f77a0459273d308917a/snapshots
./var/lib/cassandra/data/system_schema/views-9786ac1cdd583201a7cdad556410c985/snapshots
./var/lib/cassandra/data/system_schema/aggregates-924c55872e3a345bb10c12f37c1ba895/snapshots
./var/lib/cassandra/data/system_schema/keyspaces-abac5682dea631c5b535b3d6cffd0fb6/snapshots
./var/lib/cassandra/data/system_schema/tables-afddfb9dbc1e30688056eed6c302ba09/snapshots
./var/lib/cassandra/data/system_schema/columns-24101c25a2ae3af787c1b40ee1aca33f/snapshots
./var/lib/cassandra/data/system_schema/dropped_columns-5e7583b5f3f43af19a39b7e1d6f5f11f/snapshots
./var/lib/cassandra/data/system_schema/triggers-4df70b666b05325195a132b54005fd48/snapshots
./var/lib/cassandra/data/system_auth/role_permissions-3afbe79f219431a7add7f5ab90d8ec9c/snapshots
./var/lib/cassandra/data/system_auth/role_members-0ecdaa87f8fb3e6088d174fb36fe5c0d/snapshots
./var/lib/cassandra/data/system_auth/network_permissions-d46780c22f1c3db9b4c1b8d9fbc0cc23/snapshots
./var/lib/cassandra/data/system_auth/roles-5bc52802de2535edaeab188eecebb090/snapshots
./var/lib/cassandra/data/system_auth/resource_role_permissons_index-5f2fbdad91f13946bd25d5da3a5c35ec/snapshots
./var/lib/cassandra/data/system/available_ranges-c539fcabd65a31d18133d25605643ee3/snapshots
./var/lib/cassandra/data/system/batches-919a4bc57a333573b03e13fc3f68b465/snapshots
./var/lib/cassandra/data/system/peer_events_v2-0e65065fe40138ed9507b9213fae8d11/snapshots
./var/lib/cassandra/data/system/built_views-4b3c50a9ea873d7691016dbc9c38494a/snapshots
./var/lib/cassandra/data/system/sstable_activity_v2-62efe31f3be8310c8d298963439c1288/snapshots
./var/lib/cassandra/data/system/IndexInfo-9f5c6374d48532299a0a5094af9ad1e3/snapshots
./var/lib/cassandra/data/system/paxos-b7b7f0c2fd0a34108c053ef614bb7c2d/snapshots
./var/lib/cassandra/data/system/sstable_activity-5a1ff267ace03f128563cfae6103c65e/snapshots
./var/lib/cassandra/data/system/view_builds_in_progress-6c22df66c3bd3df6b74d21179c6a9fe9/snapshots
./var/lib/cassandra/data/system/compaction_history-b4dbb7b4dc493fb5b3bfce6e434832ca/snapshots
./var/lib/cassandra/data/system/peers_v2-c4325fbb8e5e3bafbd070f9250ed818e/snapshots
./var/lib/cassandra/data/system/peer_events-59dfeaea8db2334191ef109974d81484/snapshots
./var/lib/cassandra/data/system/transferred_ranges-6cad20f7d4f53af2b6e20da33c6c1f83/snapshots
./var/lib/cassandra/data/system/transferred_ranges_v2-1ff78f1a7df13a2aa9986f4932270af5/snapshots
./var/lib/cassandra/data/system/paxos_repair_history-ecb8666740b23316bb91e612c8047457/snapshots
./var/lib/cassandra/data/system/table_estimates-176c39cdb93d33a5a2188eb06a56f66e/snapshots
./var/lib/cassandra/data/system/local-7ad54392bcdd35a684174e047860b377/snapshots
./var/lib/cassandra/data/system/available_ranges_v2-4224a0882ac93d0c889dfbb5f0facda0/snapshots
./var/lib/cassandra/data/system/prepared_statements-18a9c2576a0c3841ba718cd529849fef/snapshots
./var/lib/cassandra/data/system/top_partitions-7e5a361c317c351fb15fffd8afd3dd4b/snapshots
./var/lib/cassandra/data/system/size_estimates-618f817b005f3678b8a453f3930b8e86/snapshots
./var/lib/cassandra/data/system/repairs-a3d277d1cfaf36f5a2a738d5eea9ad6a/snapshots
./var/lib/cassandra/data/system/peers-37f71aca7dc2383ba70672528af04d4f/snapshots
./var/lib/cassandra/data/system_distributed/parent_repair_history-deabd734b99d3b9c92e5fd92eb5abf14/snapshots
./var/lib/cassandra/data/system_distributed/repair_history-759fffad624b318180eefa9a52d1f627/snapshots
./var/lib/cassandra/data/system_distributed/partition_denylist-d6123acc864934969d4ef3fe39a6018b/snapshots
./var/lib/cassandra/data/system_distributed/view_build_status-5582b59f8e4e35e1b9133acada51eb04/snapshots
./var/lib/cassandra/data/system_traces/sessions-c5e99f1686773914b17e960613512345/snapshots
./var/lib/cassandra/data/system_traces/events-8826e8e9e16a372887533bc1fc713c25/snapshots
```

Создадим снимок **catalog-ks** для всех таблиц keyspace **catalogkeyspace**

```bash
nodetool snapshot --tag catalog-ks catalogkeyspace
```

```bash
root@3ab8c4be0861:/# nodetool snapshot --tag catalog-ks catalogkeyspace
Requested creating snapshot(s) for [catalogkeyspace] with snapshot name [catalog-ks] and options {skipFlush=false}
Snapshot directory: catalog-ks
```

Создание снимка для всех таблиц нескольких keyspace **catalogkeyspace** и **cqlkeyspace** (список через пробел)

```bash
nodetool snapshot --tag catalog-cql-ks catalogkeyspace cqlkeyspace
```

Создание снимка для одной таблицы из keyspace

```bash
nodetool snapshot --tag magazine --table magazine  catalogkeyspace
```

Создание снимка для нескольких таблиц из разных keyspace (список через запятую без пробелов)

```bash
nodetool snapshot --kt-list cqlkeyspace.t,cqlkeyspace.t2 --tag multi-table
```

Посмотреть список снепшотов:

```bash
nodetool listsnapshots
```

#### Поиск снепшотов

```bash
find -name snapshots
```

Результат:

```bash
./var/lib/cassandra/data/cqlkeyspace/t2-ea230eb0b84e11ee85a6ddcd8bd8c5a1/snapshots
./var/lib/cassandra/data/cqlkeyspace/t-e797cff0b84e11ee85a6ddcd8bd8c5a1/snapshots
./var/lib/cassandra/data/catalogkeyspace/journal-14c906b0b84f11ee85a6ddcd8bd8c5a1/snapshots
./var/lib/cassandra/data/catalogkeyspace/magazine-196a0610b84f11ee85a6ddcd8bd8c5a1/snapshots
```

В списке ищем нужную нам таблицу, например **journal** из keyspace **catalogkeyspace** 

```bash
 cd /var/lib/cassandra/data/catalogkeyspace/journal-14c906b0b84f11ee85a6ddcd8bd8c5a1/snapshots && ls -l
```

Получаем список наших снимков, которые мы выполняли.
Результат:

```bash
root@3ab8c4be0861:/# cd /var/lib/cassandra/data/catalogkeyspace/journal-14c906b0b84f11ee85a6ddcd8bd8c5a1/snapshots && ls -l
total 16
drwxr-xr-x 2 cassandra cassandra 4096 Jan 21 11:33 1705836798898
drwxr-xr-x 2 cassandra cassandra 4096 Jan 21 12:18 catalog-cql-ks
drwxr-xr-x 2 cassandra cassandra 4096 Jan 21 11:59 catalog-ks
drwxr-xr-x 2 cassandra cassandra 4096 Jan 21 12:13 catalog-ks1
```

Переходим в один из каталогов

```bash
cd catalog-ks && ls -l
```

Получим список файлов среди которых есть CQL скрипт для восстановления таблицы
Результат:

```bash
root@3ab8c4be0861:/var/lib/cassandra/data/catalogkeyspace/journal-14c906b0b84f11ee85a6ddcd8bd8c5a1/snapshots# cd catalog-ks && ls -l
total 44
-rw-r--r-- 1 cassandra cassandra   88 Jan 21 11:59 manifest.json
-rw-r--r-- 5 cassandra cassandra   47 Jan 21 11:33 nb-1-big-CompressionInfo.db
-rw-r--r-- 5 cassandra cassandra   99 Jan 21 11:33 nb-1-big-Data.db
-rw-r--r-- 5 cassandra cassandra   10 Jan 21 11:33 nb-1-big-Digest.crc32
-rw-r--r-- 5 cassandra cassandra   16 Jan 21 11:33 nb-1-big-Filter.db
-rw-r--r-- 5 cassandra cassandra   16 Jan 21 11:33 nb-1-big-Index.db
-rw-r--r-- 5 cassandra cassandra 4768 Jan 21 11:33 nb-1-big-Statistics.db
-rw-r--r-- 5 cassandra cassandra   56 Jan 21 11:33 nb-1-big-Summary.db
-rw-r--r-- 5 cassandra cassandra   92 Jan 21 11:33 nb-1-big-TOC.txt
-rw-r--r-- 1 cassandra cassandra  922 Jan 21 11:59 schema.cql
```

#### Очистка снепшотов

Удаление снепшотов одной таблицы из keyspace

```bash
nodetool clearsnapshot -t magazine cqlkeyspace
```

удаление всех снепшотов keyspace

```bash
nodetool clearsnapshot -all cqlkeyspace
```

### Incremental Backups

#### Настройка для инкрементального бекапа

Для создания инкрементального бекапа необходимо включить параметр **incremental_backups** в **cassandra.yaml**

```bash
incremental_backups: true
```

#### Создание инкрементального бекапа

После создания таблиц выполним очистку командой **nodetool flush**. В результате будет создан инкрементальный бекап.

```bash
nodetool flush cqlkeyspace t
nodetool flush cqlkeyspace t2
nodetool flush catalogkeyspace journal magazine
```

Список резервных копий:

```bash
find -name backups
```

### Восстановление из инкрементального бекапа и снепшотов

**Восстановление производится при помощи двух утилит:**
- sstableloader
- nodetool import

Необходимо обратить внимание:
- Cнепшот содержит, по сути, тот же набор файлов, что и инкрементная резервная копия с несколькими дополнительными файлами. 
- Снепшот включает файл schema.cql для DDL схемы для создания таблицы в CQL. 
- Резервная копия таблицы не содержит DDL, который необходимо получить из моментального снимка при восстановлении из инкрементной резервной копии.

Удалим таблицу **cqlkeyspace.t**

```sql
cqlsh> SELECT * FROM cqlkeyspace.t;

 id | k | v
----+---+------
  1 | 1 | val1
  0 | 0 | val0

(2 rows)
cqlsh> DROP cqlkeyspace.t;
SyntaxException: line 1:5 no viable alternative at input 'cqlkeyspace' ([DROP] cqlkeyspace...)
cqlsh> DELETE cqlkeyspace.t;
SyntaxException: line 1:20 mismatched input ';' expecting K_FROM (DELETE cqlkeyspace.t[;])
cqlsh> DROP TABLE cqlkeyspace.t;
cqlsh> SELECT * FROM cqlkeyspace.t;
InvalidRequest: Error from server: code=2200 [Invalid query] message="table t does not exist"
cqlsh>
```

Для восстановления ищем нужный каталог в снепшотах, восстанавливаем таблицу и данные

```bash
find -name snapshots
cd /var/lib/cassandra/data/cqlkeyspace/t-e797cff0b84e11ee85a6ddcd8bd8c5a1/snapshots && ls -l
cd multi-table && ls -l
cqlsh -f schema.cql
nodetool import cqlkeyspace t /var/lib/cassandra/data/cqlkeyspace/t-e797cff0b84e11ee85a6ddcd8bd8c5a1/snapshots/multi-table
select * from cqlkeyspace.t;
```

Результат:

```bash
root@3ab8c4be0861:/var/lib/cassandra/data/cqlkeyspace/t-e797cff0b84e11ee85a6ddcd8bd8c5a1/snapshots/multi-table# cqlsh -f schema.cql
root@3ab8c4be0861:/var/lib/cassandra/data/cqlkeyspace/t-e797cff0b84e11ee85a6ddcd8bd8c5a1/snapshots/multi-table# cqlsh
Connected to demo at 127.0.0.1:9042
[cqlsh 6.1.0 | Cassandra 4.1.3 | CQL spec 3.4.6 | Native protocol v5]
Use HELP for help.
cqlsh> SELECT * FROM cqlkeyspace.t;

 id | k | v
----+---+---

(0 rows)
cqlsh> exit

root@3ab8c4be0861:/# nodetool import cqlkeyspace t /var/lib/cassandra/data/cqlkeyspace/t-e797cff0b84e11ee85a6ddcd8bd8c5a1/snapshots/multi-table
root@3ab8c4be0861:/# cqlsh
Connected to demo at 127.0.0.1:9042
[cqlsh 6.1.0 | Cassandra 4.1.3 | CQL spec 3.4.6 | Native protocol v5]
Use HELP for help.
cqlsh> select * from cqlkeyspace.t;

 id | k | v
----+---+------
  1 | 1 | val1
  0 | 0 | val0

(2 rows)
cqlsh>
```

#### Использование sstableloader

Для начала необходимо скопировать файлы снепшота в каталог: .../\<keyspace\>/\<table\> , затем запустить утилиту:

```bash
mkdir -p /tmp/cqlkeyspace/t
cp /var/lib/cassandra/data/cqlkeyspace/t-e797cff0b84e11ee85a6ddcd8bd8c5a1/snapshots/truncated-1705849278370-t/* /tmp/cqlkeyspace/t/
sstableloader --nodes 172.23.0.4 /tmp/cqlkeyspace/t/
```

```bash
root@3ab8c4be0861:/# mkdir -p /tmp/cqlkeyspace/t
root@3ab8c4be0861:/# cp /var/lib/cassandra/data/cqlkeyspace/t-e797cff0b84e11ee85a6ddcd8bd8c5a1/snapshots/truncated-1705849278370-t/* /tmp/cqlkeyspace/t/
root@3ab8c4be0861:/# sstableloader --nodes 172.23.0.4 /tmp/cqlkeyspace/t/
WARN  15:26:50,573 Only 51.965GiB free across all data volumes. Consider adding more capacity to your cluster or removing obsolete snapshots
Established connection to initial hosts
Opening sstables and calculating sections to stream
Streaming relevant part of /tmp/cqlkeyspace/t/nb-1-big-Data.db  to [/172.23.0.4:7000, /172.23.0.3:7000, /172.23.0.2:7000]
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:1/7 1  % [/172.23.0.2:7000]0:1/7 1  % total: 0% 0.030KiB/s (avg: 0.030KiB/s)
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:1/7 1  % [/172.23.0.2:7000]0:1/7 1  % total: 0% 0.000KiB/s (avg: 0.030KiB/s)
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:2/7 1  % [/172.23.0.2:7000]0:2/7 1  % total: 1% 5.381KiB/s (avg: 0.038KiB/s)
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:2/7 1  % [/172.23.0.2:7000]0:2/7 1  % total: 1% 0.000KiB/s (avg: 0.038KiB/s)
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:2/7 1  % [/172.23.0.2:7000]0:3/7 97 % total: 32% 2.503MiB/s (avg: 1.202KiB/s)
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:2/7 1  % [/172.23.0.2:7000]0:4/7 98 % total: 33% 5.341KiB/s (avg: 1.211KiB/s)
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:2/7 1  % [/172.23.0.2:7000]0:5/7 98 % total: 33% 3.271KiB/s (avg: 1.214KiB/s)
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:2/7 1  % [/172.23.0.2:7000]0:6/7 99 % total: 33% 2.979KiB/s (avg: 1.222KiB/s)
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:2/7 1  % [/172.23.0.2:7000]0:7/7 100% total: 33% 0.854KiB/s (avg: 1.221KiB/s)
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:3/7 97 % [/172.23.0.2:7000]0:7/7 100% total: 65% 757.845KiB/s (avg: 2.369KiB/s)
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:4/7 98 % [/172.23.0.2:7000]0:7/7 100% total: 66% 3.087KiB/s (avg: 2.372KiB/s)
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:5/7 98 % [/172.23.0.2:7000]0:7/7 100% total: 66% 0.346KiB/s (avg: 2.350KiB/s)
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:6/7 99 % [/172.23.0.2:7000]0:7/7 100% total: 66% 2.287KiB/s (avg: 2.349KiB/s)
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:7/7 100% [/172.23.0.2:7000]0:7/7 100% total: 66% 1.062KiB/s (avg: 2.346KiB/s)
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:7/7 100% [/172.23.0.2:7000]0:7/7 100% total: 66% 0.000KiB/s (avg: 2.261KiB/s)
progress: [/172.23.0.4:7000]0:0/7 0  % [/172.23.0.3:7000]0:7/7 100% [/172.23.0.2:7000]0:7/7 100% total: 66% 0.000KiB/s (avg: 2.256KiB/s)
progress: [/172.23.0.4:7000]0:1/7 1  % [/172.23.0.3:7000]0:7/7 100% [/172.23.0.2:7000]0:7/7 100% total: 67% 3.822KiB/s (avg: 2.262KiB/s)
progress: [/172.23.0.4:7000]0:2/7 1  % [/172.23.0.3:7000]0:7/7 100% [/172.23.0.2:7000]0:7/7 100% total: 67% 4.226KiB/s (avg: 2.264KiB/s)
progress: [/172.23.0.4:7000]0:3/7 97 % [/172.23.0.3:7000]0:7/7 100% [/172.23.0.2:7000]0:7/7 100% total: 99% 770.407KiB/s (avg: 3.335KiB/s)
progress: [/172.23.0.4:7000]0:4/7 98 % [/172.23.0.3:7000]0:7/7 100% [/172.23.0.2:7000]0:7/7 100% total: 99% 9.256KiB/s (avg: 3.342KiB/s)
progress: [/172.23.0.4:7000]0:5/7 98 % [/172.23.0.3:7000]0:7/7 100% [/172.23.0.2:7000]0:7/7 100% total: 99% 1.377KiB/s (avg: 3.337KiB/s)
progress: [/172.23.0.4:7000]0:6/7 99 % [/172.23.0.3:7000]0:7/7 100% [/172.23.0.2:7000]0:7/7 100% total: 99% 12.692KiB/s (avg: 3.346KiB/s)
progress: [/172.23.0.4:7000]0:7/7 100% [/172.23.0.3:7000]0:7/7 100% [/172.23.0.2:7000]0:7/7 100% total: 100% 1.885KiB/s (avg: 3.344KiB/s)
progress: [/172.23.0.4:7000]0:7/7 100% [/172.23.0.3:7000]0:7/7 100% [/172.23.0.2:7000]0:7/7 100% total: 100% 0.000KiB/s (avg: 3.251KiB/s)

Summary statistics:
   Connections per host    : 1
   Total files transferred : 21
   Total bytes transferred : 14.549KiB
   Total duration          : 4723 ms
   Average transfer rate   : 3.080KiB/s
   Peak transfer rate      : 3.346KiB/s

root@3ab8c4be0861:/# cqlsh
Connected to demo at 127.0.0.1:9042
[cqlsh 6.1.0 | Cassandra 4.1.3 | CQL spec 3.4.6 | Native protocol v5]
Use HELP for help.
cqlsh> SELECT * FROM cqlkeyspace.t;

 id | k | v
----+---+------
  1 | 1 | val1
  0 | 0 | val0

(2 rows)
cqlsh>
```

### Вывод

Штатные средства хороши для восстановления таблицы, но при большом количестве таблиц и операций я сомневаюсь в быстром нахождении нужного файла для восстановления.
Восстановление болього количества таблиц потребует значительных затрат (необходимо создать таблицы, а потом импортировать данные).
