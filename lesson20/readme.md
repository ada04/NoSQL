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

```cypher
match (gl:People {name:'Gleb'}) -[*1..4]- (c:Company) return c
```

![visualisation (1)](https://github.com/ada04/NoSQL/assets/40420948/605eeb91-153c-4de3-ac7c-b4e2febb2438)

### Аэропорты/Авиакомпании

```cypher
create (a1:airline {name: 'Aeroflot', code:'AFL', base:'Moscow'})
create (r1:airline {name: 'Rossia', code:'SDM', base: 'Sankt-Petersburg'})
create (u1:airline {name: 'Utair', code: 'UTA', base:'Tumen'})
create (svo:airport {name: 'Sheremetevo', vpp: 3, iata: 'SVO', town: 'Moscow'})
create (vko:airport {name: 'Vnukovo', vpp:2, iata: 'VKO', town: 'Moscow'})
create (aer:airport {name: 'Adler', vpp:2, iata: 'AER', town: 'Sochi'})
create (ist:airport {name: 'Istambul Havalimani Camii', vpp: 5, iata:'IST', town: 'Istambul'})
create (tjm:airport {name: 'Roschino', vpp: 2, iata:'TJM', town:'Tumen'})
create (spb: airport {name: 'SPb', vpp:2, isata: 'SPB', town: 'Sankt-Petersburg'})
create (a1) -[:FLY]-> (svo)
create (a1) -[:FLY]-> (aer)
create (a1) -[:FLY]-> (ist)
create (r1) -[:FLY]-> (vko)
create (r1) -[:FLY]-> (aer)
create (r1) -[:FLY]-> (tjm)
create (u1) -[:FLY]-> (vko)
create (u1) -[:FLY]-> (tjm)
create (svo) -[:FLYGHT {airline:'AFL', price: 8500}]-> (aer)
create (svo) -[:FLYGHT {airline:'AFL', price: 22000}]-> (ist)
create (svo) -[:FLYGHT {airline:'AFL', price: 20000}]-> (tjm)
create (aer) -[:FLYGHT {airline:'AFL', price: 15000}]-> (ist)
create (tjm) -[:FLYGHT {airline:'SDM', price: 12000}]-> (vko)
create (tjm) -[:FLYGHT {airline:'UTA', price: 11000}]-> (vko)
create (vko) -[:FLYGHT {airline:'UTA', price: 10000}]-> (aer)
create (vko) -[:FLYGHT {airline:'SDM', price: 18000}]-> (ist)
create (svo) -[:FLYGHT {airline:'AFL', price: 4500}]-> (spb)
create (spb) -[:FLYGHT {airline:'AFL', price: 9500}]-> (aer)


create (aer) -[:FLYGHT {airline:'AFL', price: 8500}]-> (svo)
create (ist) -[:FLYGHT {airline:'AFL', price: 22000}]-> (svo)
create (tjm) -[:FLYGHT {airline:'AFL', price: 20000}]-> (svo)
create (ist) -[:FLYGHT {airline:'AFL', price: 15000}]-> (aer)
create (tjm) -[:FLYGHT {airline:'SDM', price: 12000}]-> (vko)
create (tjm) -[:FLYGHT {airline:'UTA', price: 11000}]-> (vko)
create (aer) -[:FLYGHT {airline:'UTA', price: 10000}]-> (vko)
create (ist) -[:FLYGHT {airline:'SDM', price: 18000}]-> (vko)

match(svo:airport {iata: 'SVO'})
match(aer:airport {iata: 'AER'})

```

  Created 8 nodes, created 24 relationships, set 61 properties, added 8 labels

![image](https://github.com/ada04/NoSQL/assets/40420948/e37c74d9-a225-4774-9151-5363320b7e8c)

![image](https://github.com/ada04/NoSQL/assets/40420948/ff1bb3bd-7f90-44f2-a729-4b058eed9a86)


```cypher
MATCH (from:airport { town:'Tumen' }), (to:airport { town: 'Sochi'}) , cost = (from)-[:FLYGHT]->(to) 
RETURN cost, 
length(cost),  
min(reduce(price = 0, r in relationships(FLYGHT) | price+r.price)) AS totalPrice
ORDER BY length(cost), totalPrice
```
