# Урок 11 - Cassandra

Масштабирование и отказоустойчивость Cassandra. Часть 1

# Домашнее задание

## Цель:
В результате выполнения ДЗ вы подготовите среду и развернете Cassandra кластер для дальнейшего изучения возможностей маштабирования и восстанавления Cassandra кластеров.

- развернуть docker локально или в облаке
- поднять 3 узловый Cassandra кластер.
- Создать keyspase с 2-мя таблицами. Одна из таблиц должна иметь составной Partition key, как минимум одно поле - clustering key, как минимум одно поле не входящее в primiry key.
- Заполнить данными обе таблицы.
- Выполнить 2-3 варианта запроса использую WHERE
- Создать вторичный индекс на поле, не входящее в primiry key.
- (*) нагрузить кластер при помощи Cassandra Stress Tool (используя "How to use Apache Cassandra Stress Tool.pdf" из материалов).

## Описание/Пошаговая инструкция выполнения домашнего задания:

### Разворачиваем docker локально

```bash
apt install docker.io
```

Проверяем...

```bash
docker run hello-world
```

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

Docker успешно установлен


### поднять 3 узловый Cassandra кластер.

### Создать keyspase с 2-мя таблицами. Одна из таблиц должна иметь составной Partition key, как минимум одно поле - clustering key, как минимум одно поле не входящее в primiry key.

### Заполнить данными обе таблицы.

### Выполнить 2-3 варианта запроса использую WHERE

### Создать вторичный индекс на поле, не входящее в primiry key.

### (*) нагрузить кластер при помощи Cassandra Stress Tool (используя "How to use Apache Cassandra Stress Tool.pdf" из материалов).
