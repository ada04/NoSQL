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

