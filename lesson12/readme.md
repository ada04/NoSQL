# Урок 12 - Elasticsearch

Основы работы с Elasticsearch, архитектура и возможности

- основные модели использования ElasticSearch;
- архитектура и компоненты ElasticSearch;
- происхождение "ELK стек" и его актуальность;
- установка и настройка ElasticSearch;
- создание и настройка индексов;
- типы запросов ElasticSearch и особенности их применения;
- Kibana - основы интерфейса и использования.

# Домашнее задание

Знакомство с ElasticSearch

## Цель:
В результате выполнения ДЗ вы научитесь разворачивать ES в AWS и использовать полнотекстовый нечеткий поиск

- Развернуть Instance ES – желательно в AWS.
- Создать в ES индекс, в нём должно быть обязательное поле text типа string
- Создать для индекса pattern
- Добавить в индекс как минимум 3 документа желательно со следующим содержанием:
   - «моя мама мыла посуду а кот жевал сосиски»
   - «рама была отмыта и вылизана котом»
   - «мама мыла раму»
- Написать запрос нечеткого поиска к этой коллекции документов ко ключу «мама ела сосиски»
- Расшарить коллекцию postman (желательно сдавать в таком формате)
- Прислать ссылку на коллекцию

## Описание/Пошаговая инструкция выполнения домашнего задания:

### Connection details

ES endpoint
```bash
https://41a2363c3b8f4fb7881f920bc14130bc.us-central1.gcp.cloud.es.io:443
```

### Создаем индекс и добавляем три документа
```
put /otus-hw

post /otus-hw/_create/1
{
   "id": "1",
   "text":"моя мама мыла посуду а кот жевал сосиски"
}

post /otus-hw/_create/2
{
   "id": "2",
   "text":"рама была отмыта и вылизана котом"
}

post /otus-hw/_create/3
{
   "id": "3",
   "text":"мама мыла раму"
}

```

### Поиск всех документов
```
get /otus-hw/_search
{
    "query":{
        "match_all":{}
    }
}
```

```
200 — OK (341 ms)

{
  "hits": {
    "hits": [
      {
        "_score": 1,
        "_id": "1",
        "_source": {
          "text": "моя мама мыла посуду а кот жевал сосиски",
          "id": "1"
        },
        "_index": "otus-hw"
      },
      {
        "_score": 1,
        "_id": "2",
        "_source": {
          "text": "рама была отмыта и вылизана котом",
          "id": "2"
        },
        "_index": "otus-hw"
      },
      {
        "_score": 1,
        "_id": "3",
        "_source": {
          "text": "мама мыла раму",
          "id": "3"
        },
        "_index": "otus-hw"
```

### запрос нечеткого поиска к этой коллекции документов ко ключу «мама ела сосиски»

```
{
    "query": {
        "match": {
            "text" : {
                "query": "мама ела сосиски",
                "fuzziness": "auto"
            }
        }
    }
}
```

```
200 — OK (248 ms)

        Skip to main content

Trial - 14 days left


D
Cloud
Deployments
… 
Elasticsearch
API console
Deployments
otus
Edit
Monitoring
Health
Logs and metrics
Performance
Elasticsearch
Snapshots
API console
Kibana
Integrations Server
Enterprise Search
Activity
Security
Features
Support
API console
Perform operations-related tasks from this console. You can run search queries, review the list of snapshots, check the health of your clusters, and more(opens in a new tab or window).

GET
/otus-hw/_search

Recent

Submit

Advanced

Press Enter to start editing.

When you're done, press Escape to stop editing.

  
200 — OK (248 ms)

{
  "hits": {
    "hits": [
      {
        "_score": 1.241674,
        "_id": "1",
        "_source": {
          "text": "моя мама мыла посуду а кот жевал сосиски",
          "id": "1"
        },
        "_index": "otus-hw"
      },
      {
        "_score": 0.5820575,
        "_id": "3",
        "_source": {
          "text": "мама мыла раму",
          "id": "3"
        },
        "_index": "otus-hw"
      },
      {
        "_score": 0.34421936,
        "_id": "2",
        "_source": {
          "text": "рама была отмыта и вылизана котом",
          "id": "2"
        },
        "_index": "otus-hw"
      }

```
