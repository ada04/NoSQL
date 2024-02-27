# Урок 16 - Redis. Часть 2

Redis

# Домашнее задание

Redis

## Цель:
В результате выполнения ДЗ вы поработаете с Redis.

## Описание/Пошаговая инструкция выполнения домашнего задания:

Необходимо:

- сохранить большой жсон (~20МБ) в виде разных структур - строка, hset, zset, list;
- протестировать скорость сохранения и чтения;
- предоставить отчет.

### Устанавливаем Redis на Ubuntu

Add the repository to the apt index, update it, and then install:

```bash
#sudo apt install lsb-release curl gpg
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
sudo apt-get update
sudo apt-get install redis
```

#### если используем docker
```bash
docker pull redis
docker run --name otus-redis -d redis
docker exec -it otus-redis redis-cli
```

Запускаем докер-контейнер
```bash
sudo docker run --name redis_cont -e ALLOW_EMPTY_PASSWORD=yes -p 6379:6379 -d redis
```

### GUI
Устанавливаем GUI `https://redis.com/redis-enterprise/redis-insight/`

### Загружаем данные

Для тестирования был взят обезличенный набор данных, содержащий информацию по получению услуг: В качестве ключа был взят код записи (id), в списках и множествах код услуги (u_id)
![image](https://github.com/ada04/NoSQL/assets/40420948/6f17c109-b404-4cf7-8932-d27d83257a6c)

Для загрузки использовался код, написанный на языке Python
```python
import json
import redis

def set_str():
    r.set(f'docs:{js["ID"]}:u_id', sid)
    r.set(f'docs:{js["ID"]}:int_id', js["INT_ID"])
    r.set(f'docs:{js["ID"]}:ext_id', js["EXT_ID"])
    r.set(f'docs:{js["ID"]}:status_date', js["STATUS_DATE"])
    r.set(f'docs:{js["ID"]}:status_name', js["STATUS_NAME"])
    r.set(f'docs:{js["ID"]}:status_desc', js["STATUS_DESC"])
    r.set(f'docs:{js["ID"]}:is_readed', js["IS_READED"])
    r.set(f'docs:{js["ID"]}:is_viewed', js["IS_VIEWED"])

def set_strj():
    r.set(f'docsj:{js["ID"]}', '{'+ f'"u_id":{sid}, "int_id":{js["INT_ID"]}, "ext_id":{js["EXT_ID"]}, "status_date":{js["STATUS_DATE"]}, "status_name":{js["STATUS_NAME"]}, "status_desc":{js["STATUS_DESC"]}, "is_readed":{js["IS_READED"]}, "is_viewed":{js["IS_VIEWED"]}' + '}')
    
def set_hash():
    r.hset(f'doch:{js["ID"]}', 'u_id', sid)
    r.hset(f'doch:{js["ID"]}', 'int_id', js["INT_ID"])
    r.hset(f'doch:{js["ID"]}', 'ext_id', js["EXT_ID"])
    r.hset(f'doch:{js["ID"]}', 'status_date', js["STATUS_DATE"])
    r.hset(f'doch:{js["ID"]}', 'status_name', js["STATUS_NAME"])
    r.hset(f'doch:{js["ID"]}', 'status_desc', js["STATUS_DESC"])
    r.hset(f'doch:{js["ID"]}', 'is_readed', js["IS_READED"])
    r.hset(f'doch:{js["ID"]}', 'is_viewed', js["IS_VIEWED"])

def set_list():
    val = '{'+ f'"id":{js["ID"]}, "int_id":{js["INT_ID"]}, "ext_id":{js["EXT_ID"]}, "status_date":{js["STATUS_DATE"]}, "status_name":{js["STATUS_NAME"]}, "status_desc":{js["STATUS_DESC"]}, "is_readed":{js["IS_READED"]}, "is_viewed":{js["IS_VIEWED"]}' + '}'
    r.rpush(f'docl:{sid}', val)

def set_hset():
    r.sadd(f'dochs:{sid}', '{'+ f'"id":{js["ID"]}, "int_id":{js["INT_ID"]}, "ext_id":{js["EXT_ID"]}, "status_date":{js["STATUS_DATE"]}, "status_name":{js["STATUS_NAME"]}, "status_desc":{js["STATUS_DESC"]}, "is_readed":{js["IS_READED"]}, "is_viewed":{js["IS_VIEWED"]}' + '}:'+str(js["ID"]))
    
def set_zset():
    val = '{'+ f'"id":{js["ID"]}, "int_id":{js["INT_ID"]}, "ext_id":{js["EXT_ID"]}, "status_date":{js["STATUS_DATE"]}, "status_name":{js["STATUS_NAME"]}, "status_desc":{js["STATUS_DESC"]}, "is_readed":{js["IS_READED"]}, "is_viewed":{js["IS_VIEWED"]}' + '}'
    r.zadd(f'doczs:{sid}', {val:js["ID"]})

    
r = redis.Redis(host='192.168.1.27', port=6379, decode_responses=True)

i = 0
with open('d:/ulk.json', encoding='utf-8') as f:
    for line in f:
        i += 1
        js = json.loads(line.replace("\\\\","\\"))
        jsk = js["EXT_ID"].split('-')
        if len(jsk) > 3:
            sid = jsk[2]
        else:
            sid = jsk[0]
        print(f'doc:{js["ID"]}')
        set_str()
        set_strj()
        set_hash()
        set_hset()
        set_zset()
        set_list()

        if i == 10000:
            break
        
print(f'{i} rows')
```

ВМ с Redis была развернута в VirtualBox с выделением 16Гб ОЗУ. Для теста было загружено 10000 "записей" в различных форматах.
В результате сделан вывод по использованию именно этого набора данных: для него хорошо подходит HASH, но совсем не подходит string или list.

