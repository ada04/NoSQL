# Урок 07 - Couchbase
[материал урока](lesson07.txt)

[Дополнительная теория](https://jazzteam.ru/technical-articles/couchbase/)

# Домашнее задание
## Цель
В результате выполнения ДЗ вы построение отказоустойчивый кластер Couchbase.
- Развернуть кластер Couchbase
- Создать БД, наполнить небольшими тестовыми данными
- Проверить отказоустойчивость

## Пошаговая инструкция выполнения домашнего задания:


### 0. Подготовительный этап

Выполнять ДЗ будем на ВМ с Ubuntu 22.04 c использованием docker-compose

Подготавливаем ВМ

```bash
sudo mkdir /u01
sudo chown vagrant:vagrant /u01
mkdir /u01/node1
mkdir /u01/node2
mkdir /u01/node3
mkdir compose
cd compose/
```

Будем поднимать три узла в следующей конфигурации:

```yaml
couchbase1:
  image: couchbase/server
  volumes:
    - /u01/couchbase/node1:/opt/couchbase/var
couchbase2:
  image: couchbase/server
  volumes:
    - /u01/couchbase/node2:/opt/couchbase/var
couchbase3:
  image: couchbase/server
  volumes:
    - /u01/couchbase/node3:/opt/couchbase/var
  ports:
    - 8091:8091
    - 8092:8092
    - 8093:8093
    - 11210:11210
```

Редактируем файл и запускаем узлы.

```bash
vi docker-compose.yml
sudo docker-compose up -d
```

Результат:
```bash
vagrant@ubuntu2204:~/compose$ sudo docker-compose up -d
Pulling couchbase1 (couchbase/server:)...
latest: Pulling from couchbase/server
527f5363b98e: Pull complete
6ba3786d7d7a: Pull complete
dc11377ffe20: Pull complete
c82198145505: Pull complete
f09217ae75dd: Pull complete
77b7fdb17526: Pull complete
dd33486bea71: Pull complete
d0724c2e18db: Pull complete
9e737c343dd2: Pull complete
4f4fb700ef54: Pull complete
0afb45900d01: Pull complete
Digest: sha256:001cae90dcf44f015e9e2ae9541d28a7ae3fa026338312d253bddef5567c061d
Status: Downloaded newer image for couchbase/server:latest
Creating compose_couchbase1_1 ... done
Creating compose_couchbase3_1 ... done
Creating compose_couchbase2_1 ... done
vagrant@ubuntu2204:~/compose$
```

Смотрим состояние узлов

```bash
docker ps
```

```bash
vagrant@ubuntu2204:~/compose$ sudo docker ps
CONTAINER ID   IMAGE              COMMAND                  CREATED              STATUS          PORTS                                                                                                                        NAMES
9ff9aa46e14e   couchbase/server   "/entrypoint.sh couc…"   About a minute ago   Up 43 seconds   8091-8097/tcp, 9123/tcp, 11207/tcp, 11210/tcp, 11280/tcp, 18091-18097/tcp                                                    compose_couchbase2_1
0683c2fdb71c   couchbase/server   "/entrypoint.sh couc…"   About a minute ago   Up 42 seconds   8094-8097/tcp, 9123/tcp, 0.0.0.0:8091-8093->8091-8093/tcp, 11207/tcp, 11280/tcp, 0.0.0.0:11210->11210/tcp, 18091-18097/tcp   compose_couchbase3_1
780279aade8b   couchbase/server   "/entrypoint.sh couc…"   About a minute ago   Up 43 seconds   8091-8097/tcp, 9123/tcp, 11207/tcp, 11210/tcp, 11280/tcp, 18091-18097/tcp                                                    compose_couchbase1_1
```

Состояние узлов можно посмотреть и через docker-compose

```bash
sudo docker-compose ps
```

```bash
vagrant@ubuntu2204:~/compose$ sudo docker-compose ps
        Name                      Command               State                                           Ports
------------------------------------------------------------------------------------------------------------------------------------------------------
compose_couchbase1_1   /entrypoint.sh couchbase-s ...   Up      11207/tcp, 11210/tcp, 11280/tcp, 18091/tcp, 18092/tcp, 18093/tcp, 18094/tcp,
                                                                18095/tcp, 18096/tcp, 18097/tcp, 8091/tcp, 8092/tcp, 8093/tcp, 8094/tcp, 8095/tcp,
                                                                8096/tcp, 8097/tcp, 9123/tcp
compose_couchbase2_1   /entrypoint.sh couchbase-s ...   Up      11207/tcp, 11210/tcp, 11280/tcp, 18091/tcp, 18092/tcp, 18093/tcp, 18094/tcp,
                                                                18095/tcp, 18096/tcp, 18097/tcp, 8091/tcp, 8092/tcp, 8093/tcp, 8094/tcp, 8095/tcp,
                                                                8096/tcp, 8097/tcp, 9123/tcp
compose_couchbase3_1   /entrypoint.sh couchbase-s ...   Up      11207/tcp, 0.0.0.0:11210->11210/tcp, 11280/tcp, 18091/tcp, 18092/tcp, 18093/tcp,
                                                                18094/tcp, 18095/tcp, 18096/tcp, 18097/tcp, 0.0.0.0:8091->8091/tcp,
                                                                0.0.0.0:8092->8092/tcp, 0.0.0.0:8093->8093/tcp, 8094/tcp, 8095/tcp, 8096/tcp,
                                                                8097/tcp, 9123/tcp

```

Смотрим журналы узлов

```bash
sudo docker-compose logs
```

```bash
Attaching to compose_couchbase2_1, compose_couchbase3_1, compose_couchbase1_1
couchbase1_1  | Starting Couchbase Server -- Web UI available at http://<ip>:8091
couchbase1_1  | and logs available in /opt/couchbase/var/lib/couchbase/logs
couchbase1_1  | Monotonic time stepped backwards!
couchbase1_1  | Previous time: 52115988258
couchbase1_1  | Current time:  52115987846
couchbase3_1  | Starting Couchbase Server -- Web UI available at http://<ip>:8091
couchbase3_1  | and logs available in /opt/couchbase/var/lib/couchbase/logs
couchbase3_1  | Monotonic time stepped backwards!
couchbase3_1  | Previous time: 93828050546
couchbase3_1  | Current time:  93828042552
couchbase2_1  | Starting Couchbase Server -- Web UI available at http://<ip>:8091
couchbase2_1  | and logs available in /opt/couchbase/var/lib/couchbase/logs
couchbase2_1  | Monotonic time stepped backwards!
couchbase2_1  | Previous time: 87982630044
couchbase2_1  | Current time:  87982617124
```

Полную информацию об узле можно посмотреть командой

```bash
sudo docker inspect  <имя узла>
```

```bash
sudo docker inspect  compose_couchbase1_1
```

### 1. Развернуть кластер Couchbase

Для настройки/создания кластера CouchBase заходим через web браузер на IP ВМ внутри которой развернут docker. В нашем случае 192.168.1.19. 
При создании конфигурации docker-compose мы указали маппинг портов (опция -p или раздел ports в yml), который нам разрешает доступ внутрь контейнера.

    http://192.168.1.19:8091 

#### 1. На первом экране заполняем данные кластера

|ClusterName:| couchb_cl|
|Admin:| admin|
|Password:| qwerty_123|

#### 2. Принимаем terms&conditions
и нажимаем `Finish With Defaults` (но можно и посмотреть настройки распределения дисков и памяти в ручном режиме)

#### 3. Заходим в раздел Servers
Добавляем сервера `172.17.0.3` и `172.17.0.4`

Запускаем `Rebalance` (Кластер необходимо перебалансировать, чтобы обеспечить правильное распределение данных между вновь добавленными или удаленными узлами. )


```bash
```

### 2. Создать БД, наполнить небольшими тестовыми данными

Заходим в раздел `Buckets` и выбираем `add simple bucket`. Загружаем тестовые данные **travel-sample**.


### 3. Проверить отказоустойчивость

```bash
```

```bash
```
