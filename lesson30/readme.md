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

[QuickStart](https://cloud.yandex.ru/ru/docs/managed-clickhouse/quickstart)

### Создание ВМ для клиентской части

В YC создвем ВМ с доступом извне и подключаемся к ней

Подключаем DEB-репозиторий

```bash
sudo apt-get install -y apt-transport-https ca-certificates dirmngr
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754
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

### Создание кластера

[Дока](https://cloud.yandex.ru/ru/docs/managed-clickhouse/operations/cluster-create)

- В консоли управления выбираем каталог, в котором нужно создать кластер БД.
- Выбираем сервис Managed Service for ClickHouse.
- Нажимаем кнопку Создать кластер.
- Задаем параметры кластера и нажимаем кнопку Создать кластер.
- Дожидаемся, когда кластер будет готов к работе: его статус на панели Managed Service for ClickHouse® сменится на Running, а состояние — на Alive. Это может занять некоторое время.

### Подключаемся к БД

Для подключения к серверу БД получаем SSL-сертификаты:

```bash
sudo mkdir --parents /usr/local/share/ca-certificates/Yandex/ && \
sudo wget "https://storage.yandexcloud.net/cloud-certs/RootCA.pem" \
     --output-document /usr/local/share/ca-certificates/Yandex/RootCA.crt && \
sudo wget "https://storage.yandexcloud.net/cloud-certs/IntermediateCA.pem" \
     --output-document /usr/local/share/ca-certificates/Yandex/IntermediateCA.crt && \
sudo chmod 655 \
     /usr/local/share/ca-certificates/Yandex/RootCA.crt \
     /usr/local/share/ca-certificates/Yandex/IntermediateCA.crt && \
sudo update-ca-certificates
```

Сертификаты будут сохранены в файлах:

    /usr/local/share/ca-certificates/Yandex/RootCA.crt
    /usr/local/share/ca-certificates/Yandex/IntermediateCA.crt

Используем для подключения ClickHouse® CLI, для этого укажем путь к SSL-сертификату RootCA.crt в конфигурационном файле, в элементе <caConfig>:

```
<config>
  <openSSL>
    <client>
      <loadDefaultCAFile>true</loadDefaultCAFile>
      <caConfig>/usr/local/share/ca-certificates/Yandex/RootCA.crt</caConfig>
      <cacheSessions>true</cacheSessions>
      <disableProtocols>sslv2,sslv3</disableProtocols>
      <preferServerCiphers>true</preferServerCiphers>
      <invalidCertificateHandler>
        <name>RejectCertificateHandler</name>
      </invalidCertificateHandler>
    </client>
  </openSSL>
</config>
```

Запустиv ClickHouse® CLI со следующими параметрами:

```bash
clickhouse-client --host <FQDN_любого_хоста_ClickHouse®> \
                  --secure \
                  --user user1 \
                  --database <имя_БД> \
                  --port 9440 \
                  --ask-password
```
