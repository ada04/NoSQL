# Урок 9 - Clickhouse

# Домашнее задание
## Цель
В результате выполнения ДЗ в разверните БД.

## Описание/Пошаговая инструкция выполнения домашнего задания:
Необходимо, используя туториал https://clickhouse.tech/docs/ru/getting-started/tutorial/ :

- развернуть БД;
- выполнить импорт тестовой БД;
- выполнить несколько запросов и оценить скорость выполнения.
- развернуть дополнительно одну из тестовых БД https://clickhouse.com/docs/en/getting-started/example-datasets , протестировать скорость запросов
- развернуть Кликхаус в кластерном исполнении, создать распределенную таблицу, заполнить данными и протестировать скорость по сравнению с 1 инстансом

### Устанавливаем сервер

Первую часть задания будем выполнять на ВМ (VirtualBox) с установленной Ubuntu 22.04.3

Следуя инструкциям устанавливаем Clickhouse:

```bash
grep -q sse4_2 /proc/cpuinfo && echo "SSE 4.2 supported" || echo "SSE 4.2 not supported"

sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates dirmngr

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754
echo "deb https://packages.clickhouse.com/deb stable main" | sudo tee \
    /etc/apt/sources.list.d/clickhouse.list

sudo apt-get update

sudo apt-get install -y clickhouse-server clickhouse-client
```

В конце установки можно посмотреть что было создано и где размещаются логи, данные и бинарники... (Полный лог установки: [Log](./out_01.log))

```bash
Creating clickhouse group if it does not exist.
 groupadd -r clickhouse
Creating clickhouse user if it does not exist.
 useradd -r --shell /bin/false --home-dir /nonexistent -g clickhouse clickhouse
Will set ulimits for clickhouse user in /etc/security/limits.d/clickhouse.conf.
Creating config directory /etc/clickhouse-server/config.d that is used for tweaks of main server configuration.
Creating config directory /etc/clickhouse-server/users.d that is used for tweaks of users configuration.
Config file /etc/clickhouse-server/config.xml already exists, will keep it and extract path info from it.
/etc/clickhouse-server/config.xml has /var/lib/clickhouse/ as data path.
/etc/clickhouse-server/config.xml has /var/log/clickhouse-server/ as log path.
Users config file /etc/clickhouse-server/users.xml already exists, will keep it and extract users info from it.
Creating log directory /var/log/clickhouse-server/.
Creating data directory /var/lib/clickhouse/.
Creating pid directory /var/run/clickhouse-server.
 chown -R clickhouse:clickhouse '/var/log/clickhouse-server/'
 chown -R clickhouse:clickhouse '/var/run/clickhouse-server'
 chown  clickhouse:clickhouse '/var/lib/clickhouse/'
 groupadd -r clickhouse-bridge
 useradd -r --shell /bin/false --home-dir /nonexistent -g clickhouse-bridge clickhouse-bridge
 chown -R clickhouse-bridge:clickhouse-bridge '/usr/bin/clickhouse-odbc-bridge'
 chown -R clickhouse-bridge:clickhouse-bridge '/usr/bin/clickhouse-library-bridge'
Enter password for default user:
Password for default user is saved in file /etc/clickhouse-server/users.d/default-password.xml.
Setting capabilities for clickhouse binary. This is optional.
 chown -R clickhouse:clickhouse '/etc/clickhouse-server'

ClickHouse has been successfully installed.

Start clickhouse-server with:
 sudo clickhouse start

Start clickhouse-client with:
 clickhouse-client --password
```

Запустим клиент и проверим все ли работает, выполнив зварос **SELECT 1**

```bash
service clickhouse-server start
clickhouse-client
ClickHouse client version 23.12.2.59 (official build).
Connecting to localhost:9000 as user default.
Password for user (default):
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 23.12.2.

Warnings:
 * Delay accounting is not enabled, OSIOWaitMicroseconds will not be gathered. Check /proc/sys/kernel/task_delayacct
 * Maximum number of threads is lower than 30000. There could be problems with handling a lot of simultaneous queries.

ubuntu2204.localdomain :) SELECT 1

SELECT 1

Query id: 5ee5fd3d-fe78-49f5-a4a9-a92552e815a6

┌─1─┐
│ 1 │
└───┘

1 row in set. Elapsed: 0.002 sec.

ubuntu2204.localdomain :) create database test1 comment 'Database for testa OTUS'

CREATE DATABASE test1
COMMENT 'Database for testa OTUS'

Query id: 2fdbefeb-8fef-4cd8-a388-7c5201a83ee7

Ok.

0 rows in set. Elapsed: 0.042 sec.

ubuntu2204.localdomain :) SELECT name, comment FROM system.databases;

SELECT
    name,
    comment
FROM system.databases

Query id: 51e29299-0cb0-41ba-bce0-e2504fe348ac

┌─name───────────────┬─comment─────────────────┐
│ INFORMATION_SCHEMA │                         │
│ default            │                         │
│ information_schema │                         │
│ system             │                         │
│ test1              │ Database for testa OTUS │
└────────────────────┴─────────────────────────┘

5 rows in set. Elapsed: 0.002 sec.

ubuntu2204.localdomain :) exit
Bye.
root@ubuntu2204:~#

```

Как видим, все запустилось успешно.

### Импортируем тестовую БД
