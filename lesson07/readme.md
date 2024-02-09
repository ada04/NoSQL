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

#### 1. Создаем новый кластер

![image](https://github.com/ada04/NoSQL/assets/40420948/4f5e9739-eb01-4ff2-9bc6-460be2d22bf0)

Заполняем данные (запоминаем имя/пароль)

ClusterName: `couchb_cl`
Admin: `admin`
Password: `qwerty_123`

#### 2. Принимаем terms&conditions
и нажимаем `Finish With Defaults` (но можно и посмотреть настройки распределения дисков и памяти в ручном режиме)

#### 3. Заходим в раздел Servers
Добавляем сервера `172.17.0.3` и `172.17.0.4`

![image](https://github.com/ada04/NoSQL/assets/40420948/adbf17c9-1402-41a9-be4a-e3791c850e24)


Запускаем `Rebalance` (Кластер необходимо перебалансировать, чтобы обеспечить правильное распределение данных между вновь добавленными или удаленными узлами. )

### 2. Создать БД, наполнить небольшими тестовыми данными

Заходим в раздел `Buckets` и выбираем `add simple bucket`. Загружаем тестовые данные **travel-sample**.

Посмотрим запросы в датасете про перевозки [Примеры](https://docs.couchbase.com/cloud/get-started/run-first-queries.html)

```sql
SELECT t.airportname, t.city 
FROM `travel-sample`.`inventory`.`airport` t
WHERE  tz = "America/Anchorage"
       AND geo.alt >= 2100;
```

```json
[
  {
    "airportname": "Anaktuvuk Pass Airport",
    "city": "Anaktuvuk Pass"
  }
]
```

```sql
SELECT route.airlineid, airline.name, route.sourceairport, route.destinationairport
FROM `travel-sample` route
INNER JOIN `travel-sample` airline
ON route.airlineid = META(airline).id
WHERE route.type = "route"
AND route.destinationairport = "SFO"
ORDER BY route.sourceairport;
```

```json
[
  {
    "airlineid": "airline_5209",
    "destinationairport": "SFO",
    "name": "United Airlines",
    "sourceairport": "ABQ"
  },
  {
    "airlineid": "airline_5209",
    "destinationairport": "SFO",
    "name": "United Airlines",
    "sourceairport": "ACV"
  },
  {
    "airlineid": "airline_5209",
    "destinationairport": "SFO",
    "name": "United Airlines",
    "sourceairport": "AKL"
  },
```

После получения результата можно посмотреть план выполнения запроса и рекомендации адвизора по созданию индексов

![image](https://github.com/ada04/NoSQL/assets/40420948/a20ec163-4627-4674-81e5-044fa9b2ca8c)

#### Проверим рекомендации... 

Последний запрос выполнялся 5.7s вернув 185 docs и 27147 bytes
Применим рекомендации и создадим индекс.
Изменим условие выборки, чтобы получить немного другой результат, исключив чение прошлого результата из кэша.

```sql
SELECT route.airlineid, airline.name, route.sourceairport, route.destinationairport
FROM `travel-sample` route
INNER JOIN `travel-sample` airline
ON route.airlineid = META(airline).id
WHERE route.type = "route"
AND route.destinationairport = "ATL"
ORDER BY route.sourceairport;
```

Запускаем запрос и получаем результат через 197.8ms выдавший 558 docs и 81616 bytes
 
**Вывод** рекомендации оказались действительно полезными в этом случае.


### 3. Проверить отказоустойчивость

Останавливаем один узел
```bash
sudo docker stop compose_couchbase1_1
```

```bash
vagrant@ubuntu2204:~$ sudo docker ps
CONTAINER ID   IMAGE              COMMAND                  CREATED        STATUS        PORTS                                                                                                                        NAMES
e74c3f88e05e   couchbase/server   "/entrypoint.sh couc…"   18 hours ago   Up 18 hours   8091-8097/tcp, 9123/tcp, 11207/tcp, 11210/tcp, 11280/tcp, 18091-18097/tcp                                                    compose_couchbase2_1
727bd1e6d417   couchbase/server   "/entrypoint.sh couc…"   18 hours ago   Up 18 hours   8091-8097/tcp, 9123/tcp, 11207/tcp, 11210/tcp, 11280/tcp, 18091-18097/tcp                                                    compose_couchbase1_1
2a651c0faa23   couchbase/server   "/entrypoint.sh couc…"   18 hours ago   Up 18 hours   8094-8097/tcp, 9123/tcp, 0.0.0.0:8091-8093->8091-8093/tcp, 11207/tcp, 11280/tcp, 0.0.0.0:11210->11210/tcp, 18091-18097/tcp   compose_couchbase3_1
vagrant@ubuntu2204:~$ sudo docker stop compose_couchbase1_1
compose_couchbase1_1
vagrant@ubuntu2204:~$ sudo docker ps
CONTAINER ID   IMAGE              COMMAND                  CREATED        STATUS        PORTS                                                                                                                        NAMES
e74c3f88e05e   couchbase/server   "/entrypoint.sh couc…"   18 hours ago   Up 18 hours   8091-8097/tcp, 9123/tcp, 11207/tcp, 11210/tcp, 11280/tcp, 18091-18097/tcp                                                    compose_couchbase2_1
2a651c0faa23   couchbase/server   "/entrypoint.sh couc…"   18 hours ago   Up 18 hours   8094-8097/tcp, 9123/tcp, 0.0.0.0:8091-8093->8091-8093/tcp, 11207/tcp, 11280/tcp, 0.0.0.0:11210->11210/tcp, 18091-18097/tcp   compose_couchbase3_1
vagrant@ubuntu2204:~$

```

#### Смотрим на состояние и запускаем перебалансировку

Один узел недоступен

![image](https://github.com/ada04/NoSQL/assets/40420948/773f962e-cb4b-4123-af95-544c2836004d)

Ожидание (failover установлен 30 сек) Запрос не выполнился

![image](https://github.com/ada04/NoSQL/assets/40420948/a57deb6c-9457-46cd-917b-cf0ce27f819c)

После отработки файловера, запрос выполняется

![image](https://github.com/ada04/NoSQL/assets/40420948/83600809-5d37-412b-9793-7d45b6f456a0)


Запустили Rebalance

![image](https://github.com/ada04/NoSQL/assets/40420948/18857d78-966b-4770-8206-dd5d082ec4ad)

![image](https://github.com/ada04/NoSQL/assets/40420948/c1257e2a-fbb0-42e4-9794-9aa7d56f0ad7)

Выполняем последний запрос и получаем тот же результат

![image](https://github.com/ada04/NoSQL/assets/40420948/5fd8d35c-b52c-4026-95a7-bae66428aa74)

Вернем узел в кластер

```bash
vagrant@ubuntu2204:~$ sudo docker start compose_couchbase1_1
compose_couchbase1_1
vagrant@ubuntu2204:~$ sudo docker ps
CONTAINER ID   IMAGE              COMMAND                  CREATED        STATUS         PORTS                                                                                                                        NAMES
e74c3f88e05e   couchbase/server   "/entrypoint.sh couc…"   18 hours ago   Up 18 hours    8091-8097/tcp, 9123/tcp, 11207/tcp, 11210/tcp, 11280/tcp, 18091-18097/tcp                                                    compose_couchbase2_1
727bd1e6d417   couchbase/server   "/entrypoint.sh couc…"   18 hours ago   Up 2 seconds   8091-8097/tcp, 9123/tcp, 11207/tcp, 11210/tcp, 11280/tcp, 18091-18097/tcp                                                    compose_couchbase1_1
2a651c0faa23   couchbase/server   "/entrypoint.sh couc…"   18 hours ago   Up 18 hours    8094-8097/tcp, 9123/tcp, 0.0.0.0:8091-8093->8091-8093/tcp, 11207/tcp, 11280/tcp, 0.0.0.0:11210->11210/tcp, 18091-18097/tcp   compose_couchbase3_1
vagrant@ubuntu2204:~$
```

![image](https://github.com/ada04/NoSQL/assets/40420948/adffa1d9-ab2e-41b8-99c7-d1ceb8aad9d5)

После ребалансировки у нас снова трехузловой кластер.





