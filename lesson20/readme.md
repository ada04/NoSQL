# Урок 20 Neo4j, часть 1

## Теория:
[Запросы](neo4j_p1.md)

[movie.txt](movie.txt)


### Практика - Создание нод и ребер (Фильм, режиссер и два актёра)
```cypher
create (:Director {name: 'Martin Scorsese'})
create (:Movie {title: 'Shutter Island'})
create (:Actor {name:'Leonardo DiCaprio'})
create (:Actor {name:'Mark Ruffalo'})
match (martin:Director {name:'Martin Scorsese'})
match (shutter:Movie {title: 'Shutter Island'})
create (martin) -[:CREATED]-> (shutter)
match (leo:Actor {name: 'Leonardo DiCaprio'} )
match (mark:Actor {name:'Mark Ruffalo'})
create (leo) -[:PLAYED_IN]-> (shutter)
create (mark) -[:PLAYED_IN]-> (shutter)
```

# Домашнее задание

Сравнение с неграфовыми БД

## Цель

В результате выполнения ДЗ вы поймете зачем и когда может понадобиться графовая база данных.

- Придумать 2-3 варианта, когда применима графовая база данных. Можно даже абзац на контекст каждого примера.
- Воспользоваться моделью, данными и командами из лекции или одним из своих примеров из пункта 1 и реализовать аналог в любой выбранной БД (реляционной или нет - на выбор). Сравнить команды.
- Написать, что удобнее было сделать в выбранной БД, а что в Neo4j и привести примеры.

## Описание/Пошаговая инструкция выполнения домашнего задания:

