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
match (shutter:Movie {title: 'Shutter Island'})
create (leo) -[:PLAYED_IN]-> (shutter)

match (shutter:Movie {title: 'Shutter Island'})
match (mark:Actor {name:'Mark Ruffalo'})
create (mark) -[:PLAYED_IN]-> (shutter)
```

```cypher
create (:Director {name:'Martin Scorsese'}) -[:CREATED]-> (shut:Movie {title:'Shutter Island'}) <-[:PLAYED_IN]- (:Actor {name: 'Leonardo DiCaprio'})

```

Удалить ноду без метки по ID:
```cypher
match (s) where ID(s) = 10 detach delete s
```

# Домашнее задание

Сравнение с неграфовыми БД

## Цель

В результате выполнения ДЗ вы поймете зачем и когда может понадобиться графовая база данных.

- Придумать 2-3 варианта, когда применима графовая база данных. Можно даже абзац на контекст каждого примера.
- Воспользоваться моделью, данными и командами из лекции или одним из своих примеров из пункта 1 и реализовать аналог в любой выбранной БД (реляционной или нет - на выбор). Сравнить команды.
- Написать, что удобнее было сделать в выбранной БД, а что в Neo4j и привести примеры.

## Описание/Пошаговая инструкция выполнения домашнего задания:

### Друзья/коллеги

```cypher
create (a1:People {name: 'Andrew'}) -[:WORKED]-> (mgu:Company {name: 'MGU'}) <-[:STUDIES]- (:People {name: 'Gleb'})
create (v1:People {name: 'Vitaly'}) -[:STUDIES]-> (osu:Company {name: 'OSU'}) <-[:STUDIES]- (a1)
create (s1:People {name: 'Sergey'}) -[:WORKED]-> (tnk:Company {name: 'TNK'}) <-[:WORKED]- (v1)
create (i1:People {name: 'Ivan'}) -[:WORKED]-> (rn:Company {name: 'RN'}) <-[:WORKED]- (s1)
create (i1) -[:STUDIES]-> (osu)
```

```cypher
match (n) return n
```

![visualisation](https://github.com/ada04/NoSQL/assets/40420948/62e2a85b-d324-433d-ba16-d11ff25033e5)

