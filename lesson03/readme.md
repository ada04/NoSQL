# Урок 3 - Базовые возможности mongodb

[материал урока](lesson03.md)


# Домашнее задание
## Цель
В результате выполнения ДЗ вы научитесь разворачивать MongoDB, заполнять данными и делать запросы.


## Пошаговая инструкция выполнения домашнего задания:

### 1. Установка MongoDB в ВМ VK Cloud
   
https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/

Будем устанавливать **MongoDB 7.0 Community Edition** на **Ubuntu 22.04**

Проверяем версию ОС:

    cat /etc/lsb-release

Устанавливаем ``gnupg`` и ``curl``

    sudo apt-get install gnupg curl

```bash
ubuntu@srv-mongodb-01:~$ sudo apt-get install gnupg curl
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
curl is already the newest version (7.81.0-1ubuntu1.13).
gnupg is already the newest version (2.2.27-3ubuntu2.1).
gnupg set to manually installed.
0 upgraded, 0 newly installed, 0 to remove and 98 not upgraded.
```

Импортируем MongoDB public GPG key с `https://pgp.mongodb.com/server-7.0.asc`

    curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
     sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
     --dearmor

Создаём список файлов `/etc/apt/sources.list.d/mongodb-org-7.0.list` для нашей версии Ubuntu.

    echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

Обновляем локальную базу пакетов

    sudo apt-get update

Запускаем установку последней версии **MongoDB**

    sudo apt-get install -y mongodb-org

```bash
ubuntu@srv-mongodb-01:~$ sudo apt-get install -y mongodb-org
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  mongodb-database-tools mongodb-mongosh mongodb-org-database mongodb-org-database-tools-extra mongodb-org-mongos mongodb-org-server
  mongodb-org-shell mongodb-org-tools
The following NEW packages will be installed:
  mongodb-database-tools mongodb-mongosh mongodb-org mongodb-org-database mongodb-org-database-tools-extra mongodb-org-mongos mongodb-org-server
  mongodb-org-shell mongodb-org-tools
0 upgraded, 9 newly installed, 0 to remove and 104 not upgraded.
Need to get 160 MB of archives.
After this operation, 524 MB of additional disk space will be used.
Get:1 https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0/multiverse amd64 mongodb-database-tools amd64 100.8.0 [50.7 MB]
...
...
No VM guests are running outdated hypervisor (qemu) binaries on this host.
ubuntu@srv-mongodb-01:~$

```

### 2. Запуск MongoDB Community Edition

**Data Directory** `/var/lib/mongodb`

**Log Directory** `/var/log/mongodb`

**Configuration file** `/etc/mongod.conf`

      cat /etc/mongod.conf

**Если хотим подключаться напрямую с удалённого хоста, необходимо изменить конфигурационный файл (можно сделать на любом этапе)

     sudo mv /etc/mongod.conf /etc/mongod.old
     sudo sh -c "sed 's/  bindIp: 127.0.0.1/#  bindIp: 127.0.0.1\n  bindIp: 0.0.0.0/' /etc/mongod.old > /etc/mongod.conf"

**Запуск MongoDB**

      sudo systemctl start mongod

Если появляется ошибка `Failed to start mongod.service: Unit mongod.service not found.` то 

      sudo systemctl daemon-reload

Проверяем, что MongoDB запустилась

      sudo systemctl status mongod

```bash
ubuntu@srv-mongodb-01:~$ sudo systemctl status mongod
● mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; disabled; vendor preset: enabled)
     Active: active (running) since Thu 2023-10-12 14:03:43 UTC; 41s ago
       Docs: https://docs.mongodb.org/manual
   Main PID: 2316 (mongod)
     Memory: 72.5M
        CPU: 1.208s
     CGroup: /system.slice/mongod.service
             └─2316 /usr/bin/mongod --config /etc/mongod.conf

Oct 12 14:03:43 srv-mongodb-01 systemd[1]: Started MongoDB Database Server.
Oct 12 14:03:43 srv-mongodb-01 mongod[2316]: {"t":{"$date":"2023-10-12T14:03:43.477Z"},"s":"I",  "c":"CONTROL",  "id":7484500, "ctx":"main","msg":"En>
lines 1-12/12 (END)
```

Настраиваем автозапуск сервиса MongoDB

      sudo systemctl enable mongod

Останов сервера MongoDB

      sudo systemctl stop mongod

Перезапуск сервера MongoDB

      sudo systemctl restart mongod

Просмотреть ошибки, сообщения можно в логе:

      sudo tail -n 100 /var/log/mongodb/mongod.log

Подключиться к MongoDB можно консольной утилитой `mongosh` (порт по умолчанию 27017)

     mongosh

```bash
ubuntu@srv-mongodb-01:~$ mongosh
Current Mongosh Log ID: 6527ff00777d6f965d24ad69
Connecting to:          mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.0.1
Using MongoDB:          7.0.2
Using Mongosh:          2.0.1

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

------
   The server generated these startup warnings when booting
   2023-10-12T14:03:43.766+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
   2023-10-12T14:03:44.846+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
   2023-10-12T14:03:44.846+00:00: vm.max_map_count is too low
------

test>
```

### 3. Заполнение данными БД MongoDB

Для импорта данных необходимо установить [DatabaseTools] (https://www.mongodb.com/docs/database-tools/installation/installation/)

      sudo dpkg -l mongodb-database-tools

Создаём пользователя для подключения к Mongo 

      db.createUser( { user: "dba", pwd: "otus", roles: [ "userAdminAnyDatabase", "dbAdminAnyDatabase", "readWriteAnyDatabase" ] } )
      
```sql
test> use admin
switched to db admin
admin> db.createUser( { user: "dba", pwd: "otus", roles: [ "userAdminAnyDatabase", "dbAdminAnyDatabase", "readWriteAnyDatabase" ] } )
{ ok: 1 }
```

Строка подключения для текущей конфигурации выглядит так:

      mongodb://dba:otus@79.137.175.48:27017/admin


### 4. Написание запросов на выборку и обновление данных


### 5. Создание индексов и сравннение производительности.

### 6. Удаление MongoDB

Останавливаем службу

      sudo service mongod stop

Удаляем пакеты

      sudo apt-get purge mongodb-org*

Удаляем каталоги

      sudo rm -r /var/log/mongodb
      sudo rm -r /var/lib/mongodb

