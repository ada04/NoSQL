# Урок 27 - Архитектура Tarantool 



# Домашнее задание

Работа с tarantool

## Цель:
В результате выполнения ДЗ вы поработаете с Tarantool.

## Описание/Пошаговая инструкция выполнения домашнего задания:

- Установить Tarantool Cartridge CLI
- Создать шаблон приложения командой:
    cartridge create --name myapp
- Собрать и запустить приложение:
    cartridge build
    cartridge start
- Задать любую топологию кластера в UI и сделать bootstrap


### Устанавливаем Tarantool на Ubuntu

#### Docker образ
Основной образ tarantool собран на основе alpine:3.15

```bash
docker pull tarantool/tarantool
docker run -it --rm tarantool/tarantool
```

После установки докер-контейнера мы попадаем сразу в консоль Tarantool

#### Установка из репозитория
Добавим репозиторий
```bash
curl -L https://tarantool.io/BraKbmW/release/3/installer.sh | bash
```

Усановка tarantool из репозитория (вдруг пригодится)
```bash
sudo apt-get -y install tarantool
```

Устанавливает ttCLI
```bash
apt-get install tt
```

##### Подготавливаем настройки
 
```bash
mkdir instances.enabled
cd instances.enabled/
mkdir create_db
cd create_db/
echo "instance001:" > instances.yaml
touch config.yaml
vi config.yaml
```

config.yaml
```yaml
groups:
   group001:
     replicasets:
       replicaset001:
         instances:
           instance001:
             iproto:
               listen:
               - uri: '127.0.0.1:3301'
```

##### Запуск экземпляра tarantool (если не в докере)

Из директории с созданными файлами нстароек запускаем

```bash
tt start create_db
```

Просмотр статуса БД
```bash
tt status create_db
```

Подключаемся к БД
```bash
tt connect create_db:instance001
```


### Создаем БД:

```bash
```



Создаем space с именем `bands`
```bash
box.schema.space.create('bands')
```

"Отформатируем" созданный space указав имена и типы полей
```bash
box.space.bands:format({
    { name = 'id', type = 'unsigned' },
    { name = 'band_name', type = 'string' },
    { name = 'year', type = 'unsigned' }
})
```

Создалим `primary` индекс
```bash
box.space.bands:create_index('primary', { type = "tree", parts = { 'id' } })
```


```bash
```


```bash
```


```bash
```


```bash
```




Вывод:

```bash
tarantool> box.schema.space.create('bands')
---
- engine: memtx
  before_replace: 'function: 0x7f82b9020618'
  field_count: 0
  is_sync: false
  on_replace: 'function: 0x7f82b9020458'
  temporary: false
  index: []
  is_local: false
  enabled: false
  name: bands
  id: 512
- created
...

tarantool> box.space.bands:format({
    { name = 'id', type = 'unsigned' },
    { name = 'band_name', type = 'string' },
    { name = 'year', type = 'unsigned' }
})
---
...

tarantool> box.space.bands:create_index('primary', { type = "tree", parts = { 'id' } })
---
- unique: true
  parts:
  - type: unsigned
    is_nullable: false
    fieldno: 1
  hint: true
  id: 0
  type: TREE
  space_id: 512
  name: primary
...

tarantool>

```
