# Mongodb. Базовые возможности.

1. Базовые возможности. Запустим MongoDB и поработаем с ней CRUD.
2. Шардинг + репликацию.
3. Возможности аггегационного фреймворка.
4. Оптимизации. Индексы и т.д.

Мои вопросы.

- Какой опыт работы с базами данными?

## Почему MongoDB основные приемущества?

Минусы - это продолжение наших плюсов.

- Schemeless база данных. Мы не будем прописывать схему.
- Документоориентированная. Структура в виде документа, иерархическая.
- Императивная база данных.
- Хорошо маштабируется из коробки - шардированние удобное и репликации.
- Это база данных имеет доступность.
- MongoDB имеет хороший движок работы с геоданных.
- MongoDB они использует BJSON(JSON бинанрый)
- MongoDB есть gridFS - хранить файлы до 15 Мб.

Декларативный подход и императивных подход в IT.
В декларативном подходе мы отвечаем на вопрос что? и описываем их в виде текста(SQL запроса).
В императивном подходе мы отвечаем на вопрос как? и описываем последовательность действий(вызываем функции или последовательность функций).

Если мы хотим постороить real-time приложение.
Также хорошо подходит для работы с geo данным.
Хотите дать быстрый старт проекту.

2009 году появилось.

Уходить в веб -> уходим в язык Javascript -> уходим на фронтенде -> формат JSON -> платформа NodeJS(бэкенд часть) -> не использовать которая использует Javascript и BJSON.

ObjectId - это объект, который создает уникальный ключ, который имеет строковое представление.

_id: ObjectId - это является id документа.

Сейчас актуальная версия - 7.

Как подключиться:

- mongosh
- mongo Compass GUI
- robo3t GUI.


Подключиться:
mongosh --port 27018

show dbs - это показать все базы данных.
use sample_mflix - я перешёл в базу данных sampe_mflix

db.getCollection('movies').find({}) - это SELECT * FROM movies
db.getCollection('movies').find({}, { plot: 1, title: 1 }) - SELECT plot, title FROM movies
db.getCollection('movies').find({}, { _id: 0, plot: 0 }) - исключая поля _id и plot.
db.getCollection('movies').find({ year: 2000 }) - SELECT * FROM movies WHERE year = 2000
db.getCollection('movies').find({ 'tomatoes.critic.rating': { $gt: 8 } }) - SELECT * FROM movies WHERE year = 2000
db.getCollection('movies').find({ year: { $in: [2000, 2001]} }) - SELECT * FROM movies WHERE year IN (2000, 2001)
find().limit(10) - лимит

tomatoes.critic.rating > 8
$gt - greater than

db.getCollection('movies')
    .find({ $and: [{ year: { $gte: 2000 } }, { 'imdb.rating': { $gte: 8 } } ]})
    .sort({ 'imdb.rating': -1 })
    .limit(1)

// GroupBy
{
    movies: [] // 10 фильмов
    count: 2500 // все фильмы
}

SELECT * from movies
WHERE year >= 2000 AND imdb_rating >= 8


db.getCollection('movies')
    .find({ $and: [{ year: { $gte: 2000 } }, { 'imdb.rating': { $gte: 8 } } ]})

// distict
db.getCollection('movies').distinct("year", { 'imdb.rating': { $gte: 9 } })

// Сделать один запрос
// Потом с этими данными что-то сделать и сделать другой

https://www.mongodb.com/try/download/tools - тулзы

## Загрузка бэкапа

https://github.com/neelabalan/mongodb-sample-dataset

## find

1. find() - просто поиск
2. find({ field: <value> }) - поиск по полю
3. find({ 'path.to.field': <value> }) - поиск по вложенному полю
4. .find({ age: { $gte: 18 } }) - поиск с условием(gte - greater than or equal)
5. .find({ age: { $in: [18,19] } }) - поиск с условием(in - greater than or equal)
6. find({}, { name: 1, age: 1 }) - проекция
7. Сортировка
	find().sort({ age: 1 }) // Ascending 
	find().sort({ age: -1 }) // Descending
8. find().limit(10) - лимит
9. find().skip(10) - пропустить
10. find({ $and: [{ age: { $gte: 18 } }, { city: "New York" }] }) - условие $and https://www.mongodb.com/docs/manual/reference/operator/query/and/

## Count

countDocuments

## insert

1. insertOne({ name: "Alice", age: 25, grade: "A" }) - один документ
2. insertMany([ { field1: value1, field2: value2, ... }, { field1: value3, field2: value4, ... } ]) - много документов.

## update

// Может просто заменить объект, один документ на другой
// Такое поведение + updateMany - 
// updateOne(query, newDoc) - query - селектор поиск, newDoc - это что мы хотим вставить
// updateMany

## delete

1. deleteOne({ name: "Alice", age: 25, grade: "A" }) - один документ
2. deleteMany([ { field1: value1, field2: value2, ... }, { field1: value3, field2: value4, ... } ]) - много документов.

// replaceOne, replaceMany

// bulkWrite

// MongoDB operation log

## Index

// db.getCollection('moview').createIndex({ title: 1 }, { unique: 1 })


## ObjectId

`ObjectId` в MongoDB — это 12-байтовый идентификатор, обычно используемый в качестве значения по умолчанию для поля `_id` в документах. Значения `ObjectId` автоматически генерируются MongoDB при вставке нового документа в коллекцию, и они предназначены для глобальной уникальности в пределах кластера MongoDB. Вот некоторые ключевые характеристики и детали `ObjectId`:

1. **Структура из 12 байтов**: `ObjectId` представляет собой 12-байтовое двоичное значение, обычно представленное как строка из 24 символов в шестнадцатеричной системе счисления. Структура `ObjectId` следующая:
    
    - 4 байта временной метки, представляющей момент создания ObjectId, измеряется в секундах с Unix-эпохи.
    - 5 байтов случайного значения, генерируемого один раз для каждого процесса.
    - 3 байта инкрементирующегося счетчика, инициализированного случайным значением.
2. **Уникальность**: Значения `ObjectId` предназначены для обеспечения уникальности в пределах базы данных MongoDB. Однако важно отметить, что уникальность гарантируется только в пределах одной коллекции, а не между несколькими коллекциями или базами данных.
    
3. **Временная метка создания**: Первые 4 байта `ObjectId` представляют собой временную метку, которую можно извлечь, чтобы определить момент создания документа.
    
4. **Случайность**: Следующие 5 байтов генерируются случайным образом, что делает маловероятным создание двумя разными машинами одинаковых `ObjectId` в одно и то же время.
    
5. **Инкрементирующийся счетчик**: Последние 3 байта — это инкрементирующийся счетчик. В случае создания нескольких документов в одну секунду на одной и той же машине этот счетчик гарантирует уникальность.

// Сначала создать юзера
// Создается рутовый юзер в базе админ
// Потом включаем авторизацию
// Потом создаем юзеров до бд

## Авторизация

https://www.digitalocean.com/community/tutorials/how-to-secure-mongodb-on-ubuntu-20-04-ru

# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: /var/lib/mongodb
#  engine:
#  wiredTiger:

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1 // 0.0.0.0 - открыть всем хостам.


# how the process runs
processManagement:
  timeZoneInfo: /usr/share/zoneinfo

security:
  authorization: "enabled"

#operationProfiling:

#replication:

#sharding:

## Enterprise-Only Options:

#auditLog:

#snmp:


mongodump - для бекапа
mongorestore - для восстановление
mongoimport
