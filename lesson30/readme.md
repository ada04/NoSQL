# Урок 30 Nosql в Яндекс облаке 

## Практика

[практика](NoSQL_Yandex_cloud.md)

# Домашнее задание

Облака

## Цель

В результате выполнения ДЗ вы поработает с облаками.

Необходимо:

- одну из облачных БД заполнить данными (любыми из предыдущих дз);
- протестировать скорость запросов.

Задание повышенной сложности*

- сравнить 2-3 облачных NoSQL по скорости загрузки данных и времени выполнения запросов.

 
## Описание/Пошаговая инструкция выполнения домашнего задания:

В YC создвем ВМ с доступом извне и подключаемся к ней

Подключаем DEB-репозиторий

```bash
sudo apt update && sudo apt install --yes apt-transport-https ca-certificates dirmngr && \
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD******** && \
echo "deb https://packages.clickhouse.com/deb stable main" | sudo tee \
/etc/apt/sources.list.d/clickhouse.list
```

Устанавливаем зависимости и клиентское приложение clickhouse-client

```bash
sudo apt update && sudo apt install --yes clickhouse-client
```

Загружаем файл конфигурации для clickhouse-client

```bash
mkdir -p ~/.clickhouse-client && \
wget "https://storage.yandexcloud.net/doc-files/clickhouse-client.conf.example" \
  --output-document ~/.clickhouse-client/config.xml
```
