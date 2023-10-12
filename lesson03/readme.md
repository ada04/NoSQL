## Урок 3 - Базовые возможности mongodb
**материала урока**


## Домашнее задание
### Цель
В результате выполнения ДЗ вы научитесь разворачивать MongoDB, заполнять данными и делать запросы.


### Пошаговая инструкция выполнения домашнего задания:

#### 1. Установка MongoDB в ВМ VK Cloud
   
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

#### 2. Запуск MongoDB Community Edition


#### 2. Ззаполнение данными БД MongoDB

#### 3. Написание запросов на выборку и обновление данных

#### 4. Создание индексов и сравннение производительности.
