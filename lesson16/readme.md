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

