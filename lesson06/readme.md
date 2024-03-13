# Урок 06 - Оптимизация производительности mongodb
[материал урока](lesson06.txt)


# Домашнее задание
## Цель
В результате выполнения ДЗ вы настроите реплицирование и шардирование, аутентификацию в кластере и проверите отказоустойчивость.

- построить шардированный кластер из 3 кластерных нод( по 3 инстанса с репликацией) и с кластером конфига(3 инстанса);
- добавить балансировку, нагрузить данными, выбрать хороший ключ шардирования, посмотреть как данные перебалансируются между шардами;
- поронять разные инстансы, посмотреть, что будет происходить, поднять обратно. Описать что произошло.
- настроить аутентификацию и многоролевой доступ;

## Пошаговая инструкция выполнения практики:

### 1. Установка MongoDB в ВМ VK Cloud ещё на два сервера (02 и 03)
   
https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu/

Будем устанавливать **MongoDB 7.0 Community Edition** на **Ubuntu 22.04**

Проверяем версию ОС:

    cat /etc/lsb-release



--ЯО
```
yc compute instance create \
  --name mongo-instance \
  --hostname mongo-instance \
  --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts \
  --network-interface subnet-name=default-ru-central1-b,nat-ip-version=ipv4 \
  --zone ru-central1-b \
  --metadata-from-file ssh-keys=/home/ada/.ssh/id_rsa.pub
```
