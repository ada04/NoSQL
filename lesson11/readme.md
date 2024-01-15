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

### поднять 3 узловый Cassandra кластер.

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


### Создать keyspase с 2-мя таблицами. Одна из таблиц должна иметь составной Partition key, как минимум одно поле - clustering key, как минимум одно поле не входящее в primiry key.

### Заполнить данными обе таблицы.

### Выполнить 2-3 варианта запроса использую WHERE

### Создать вторичный индекс на поле, не входящее в primiry key.

### (*) нагрузить кластер при помощи Cassandra Stress Tool (используя "How to use Apache Cassandra Stress Tool.pdf" из материалов).
