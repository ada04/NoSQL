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

Создаём пользователя для подключения к Mongo (необязательно, если подключение будет только локальное)

      db.createUser( { user: "dba", pwd: "otus", roles: [ "userAdminAnyDatabase", "dbAdminAnyDatabase", "readWriteAnyDatabase" ] } )
      
```sql
test> use admin
switched to db admin
admin> db.createUser( { user: "dba", pwd: "otus", roles: [ "userAdminAnyDatabase", "dbAdminAnyDatabase", "readWriteAnyDatabase" ] } )
{ ok: 1 }
```

Строка подключения для текущей конфигурации выглядит так:

      mongodb://dba:otus@79.137.175.48:27017/admin

**Загружаем файл с демо данными**

Производить загрузку данных будем на сервере. Для начала загрузим архивы с данными.

      wget http://data.insideairbnb.com/united-states/ma/cambridge/2023-09-23/data/listings.csv.gz http://data.insideairbnb.com/united-states/ma/cambridge/2023-09-23/data/calendar.csv.gz http://data.insideairbnb.com/united-states/ma/cambridge/2023-09-23/data/reviews.csv.gz

Разархивируем все архивы

      gunzip listings.csv.gz calendar.csv.gz reviews.csv.gz

И запустим импорт данных в MongoDB

```bash
mongoimport -d mydb -c listings --type csv --file listings.csv --headerline
mongoimport -d mydb -c calendar --type csv --file calendar.csv --headerline
mongoimport -d mydb -c reviews --type csv --file reviews.csv --headerline
```

Результат импорта должен выглядеть примерно так:
```bash
ubuntu@srv-mongodb-01:/nfs$ mongoimport -d mydb -c listings --type csv --file listings.csv --headerline
2023-10-13T13:10:29.684+0000    connected to: mongodb://localhost/
2023-10-13T13:10:30.014+0000    1054 document(s) imported successfully. 0 document(s) failed to import.

ubuntu@srv-mongodb-01:/nfs$ mongoimport -d mydb -c calendar --type csv --file calendar.csv --headerline
2023-10-13T13:11:13.765+0000    connected to: mongodb://localhost/
2023-10-13T13:11:16.770+0000    [######..................] mydb.things  4.88MB/17.8MB (27.4%)
2023-10-13T13:11:19.766+0000    [##############..........] mydb.things  10.7MB/17.8MB (59.8%)
2023-10-13T13:11:22.766+0000    [######################..] mydb.things  17.0MB/17.8MB (95.5%)
2023-10-13T13:11:23.126+0000    [########################] mydb.things  17.8MB/17.8MB (100.0%)
2023-10-13T13:11:23.126+0000    384710 document(s) imported successfully. 0 document(s) failed to import.
ubuntu@srv-mongodb-01:/nfs$ mongoimport -d mydb -c reviews --type csv --file reviews.csv --headerline
2023-10-13T13:11:35.960+0000    connected to: mongodb://localhost/
2023-10-13T13:11:37.883+0000    54870 document(s) imported successfully. 0 document(s) failed to import.
ubuntu@srv-mongodb-01:/nfs$
```

Проверим загруженные данные:

```sql
ubuntu@srv-mongodb-01:/nfs$ mongosh
Current Mongosh Log ID: 652945b067fa6762ad8db3be
Connecting to:          mongodb://127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&appName=mongosh+2.0.1
Using MongoDB:          7.0.2
Using Mongosh:          2.0.1

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

------
   The server generated these startup warnings when booting
   2023-10-13T12:22:07.925+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
   2023-10-13T12:22:10.664+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
   2023-10-13T12:22:10.664+00:00: vm.max_map_count is too low
------

test> show dbs
admin   180.00 KiB
config   72.00 KiB
local    72.00 KiB
mydb     25.08 MiB
test> use mydb
switched to db mydb
mydb> show collections
calendar
listings
reviews
mydb> db.reviews.find().limit(1)
[
  {
    _id: ObjectId("6529457af9d4a27ad6f89b99"),
    listing_id: 8521,
    id: 6009,
    date: '2009-07-23',
    reviewer_id: 25629,
    reviewer_name: 'Mara',
    comments: 'This is a fabulous apartment!  Great neighborhood, easy walk to Fresh Pond, great bread, and shops!  The apartment itself is gorgeous, light-filled, and Marc and Janet are wonderful hosts!  My son and I greatly enjoyed our short stay here and highly recommend these hosts, this apartment.'
  }
]
mydb>
```

### 4. Написание запросов на выборку и обновление данных

**Выборка всех значений коллекции**

```
db.reviews.find()
```

или

```
db.getCollection('reviews')
```

Результат этих запросов одинаковый
![image](https://github.com/ada04/NoSQL/assets/40420948/4eeae7a4-9532-45a3-9076-4c49b45ecf20)

**SELECT * from listings***

      db.listings.find()

**SELECT id, name, bedrooms, beds, price, review_scores_rating from listings**

      db.listings.find({}, {id: 1, name: 1, bedrooms: 1, beds: 1, price: 1, review_scores_rating: 1})

Если 1 заменить на 0, то это будет означать исключение этих столбцов из запроса

      db.reviews.find({}, {id: 0, reviewer_id: 0}).limit(1)

**SELECT id, name, bedrooms, beds, price, review_scores_rating from listings where review_scores_rating = 5**

      db.listings.find({review_scores_rating: 5}, {id: 1, name: 1, bedrooms: 1, beds: 1, price: 1, review_scores_rating: 1}).limit(1)

**SELECT id, name, bedrooms, beds, price, review_scores_rating from listings where review_scores_rating = 5**

      db.listings.find({review_scores_rating: { $gt: 4.9}}, {id: 1, name: 1, bedrooms: 1, beds: 1, price: 1, review_scores_rating: 1}).limit(2)

**SELECT id, name, bedrooms, beds, price, review_scores_rating from listings where bedrooms in (4, 5)**

      db.listings.find({bedrooms: { $in: [4, 5]}}, {id: 1, name: 1, bedrooms: 1, beds: 1, price: 1, review_scores_rating: 1}).limit(2)

**SELECT id, name, bedrooms, beds, price, review_scores_rating from listings where bedrooms in (4, 5) and review_scores_rating > 4,95**

      db.listings.find({ $and: [{bedrooms: { $in: [4, 5]}}, {review_scores_rating: { $gt: 4.95}}]}, {id: 1, name: 1, bedrooms: 1, beds: 1, price: 1, review_scores_rating: 1}).limit(2)

**SELECT id, name, bedrooms, beds, price, review_scores_rating from listings where bedrooms in (4, 5) and review_scores_rating > 4,95 order by price desc**

      db.listings.find({ $and: [{bedrooms: { $in: [4, 5]}}, {review_scores_rating: { $gt: 4.95}}]}, {id: 1, name: 1, bedrooms: 1, beds: 1, price: 1, review_scores_rating: 1}).sort({ 'price': -1}).limit(2)

**select count(*) from listings where review_scores_rating > 4,95**

      db.listings.find({review_scores_rating: { $gt: 4.95}}).count()

#### Insert

Смотрим состояние до вставки:

      db.reviews.find({ $and:  [{date: '2016-07-17'}, {listing_id: 8521}]})

Вставляем один документ

      db.reviews.insertOne( { listing_id: 8521, date: '2016-07-17', reviewer_id: 1, reviewer_name: 'Otus', comments: "Hi, I'm Otus!"})

Вставляем несколько документов

      db.reviews.insertMany( [{ listing_id: 8521, date: '2016-07-17', reviewer_id: 2, reviewer_name: 'Otus2', comments: "Hi, I'm Otus2!"}, { listing_id: 8521, date: '2016-07-17', reviewer_id: 3, reviewer_name: 'Otus3', comments: "Hi, I'm Otus3!"} ] )

[Log](./ins_log.txt)

#### Delete

Удаляем один документ

      db.reviews.deleteOne( { listing_id: 8521, date: '2016-07-17', reviewer_id: 1, reviewer_name: 'Otus'})

Удаляем несколько документов

      db.reviews.deleteMany( { reviewer_id: { $in: [2, 3] } })
      
[Log](./del_log.txt)

### 5. Создание индексов и сравннение производительности.

Сравнивались три запроса по коллекции `calendar`.
В первом случае выполнялась фильтрация по полю `available` (значения "f" or "t") 

      db.calendar.find({ available: 'f'}).explain("executionStats")
      
во втором по полю `date`.

      db.calendar.find({ $and:  [{date:{ $gt: '2023-10-15'}}, {date:{ $lt: '2023-11-01'}}]}).explain("executionStats")

в третьем двойное условие по дате и минимальному количеству ночей

      db.calendar.find({ $and:  [{date:{ $gt: '2023-10-15'}}, {minimum_nights: 1}]}).explain("executionStats")

В первом варианте запрос без индекса оказался выгоднее (выполнился за **174** мсек против **267** мсек с индексом)

Во втором варианте запрос с индексом оказался выгоднее (выполнился за **58** мсек против **202** мсек без индекса)

В третьем варианте запрос с индексом то же оказался выгоднее (выполнился за **189** мсек против **884** мсек без индекса)

В результате можно сделать предварительный вывод, что не все индексы приводят к ускорению запроса. 
В данном случае индекс по полю типа Дата и number привел к ускорению запроса, а по логическому тьипу, наоборот. Связано это может быть и с тем, что выбор логического типа и так происходит быстро, т.к. вариантов значений всего 2.

Мало того, в третьем варианте при сложном условии на первом запуске был индекс по полю `date`, который и использовался, но при создании второго индекса по второму полю условия, оптимизатор посчитал его более выгодным и использовал его.

Логи выполнения запросов можно посмотреть по ссылкам:


[Вариант 1](./idx_calendar.md)

[Вариант 2](./idx_calen_date.md)

[Вариант 3](./idx_calen_night.md)



### 6. Удаление MongoDB

Останавливаем службу

      sudo service mongod stop

Удаляем пакеты

      sudo apt-get purge mongodb-org*

Удаляем каталоги

      sudo rm -r /var/log/mongodb
      sudo rm -r /var/lib/mongodb

