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
create (spb:airport {name: 'SPb', vpp:2, isata: 'SPB', town: 'Sankt-Petersburg'})
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
create (tjm) -[:FLYGHT {airline:'AFL', price: 20000}]-> (svo)
create (aer) -[:FLYGHT {airline:'AFL', price: 15000}]-> (ist)
create (tjm) -[:FLYGHT {airline:'SDM', price: 12000}]-> (vko)
create (tjm) -[:FLYGHT {airline:'UTA', price: 11000}]-> (vko)
create (vko) -[:FLYGHT {airline:'UTA', price: 10000}]-> (aer)
create (vko) -[:FLYGHT {airline:'SDM', price: 18000}]-> (ist)
create (svo) -[:FLYGHT {airline:'AFL', price: 4500}]-> (spb)
create (spb) -[:FLYGHT {airline:'AFL', price: 9500}]-> (aer)
```

Created 9 nodes, created 18 relationships, set 53 properties, added 9 labels

![image](https://github.com/ada04/NoSQL/assets/40420948/28f82d28-d109-4d2a-8f45-a30c09b1c24f)

```cypher
MATCH (from:airport { town:'Tumen' }), (to:airport { town: 'Sochi'}) , path = (from)-[:FLYGHT *1..5]->(to) 
RETURN path, length(path)
```

![image](https://github.com/ada04/NoSQL/assets/40420948/fa8796da-d347-4475-9f05-8e65ea3e588c)

Попытка посчитать стоимость... что-то пошло не так...

```cypher
MATCH (from:airport { town:'Tumen' }), (to:airport { town: 'Sochi'}) , cost = (from)-[:FLYGHT]->(to) 
RETURN cost, 
length(cost),  
min(reduce(price = 0, r in relationships(FLYGHT) | price+r.price)) AS totalPrice
ORDER BY length(cost), totalPrice
```

### Сравенение с RDBMS

Создаем структуру, аналогичную второму примеру (используем СУБД Oracle)
```sql
create table ada.airline (code varchar2(5), name varchar2(20), base varchar2(50),
constraint airline_pk primary key (code));

create table ada.airport (iata varchar2(5), name varchar2(50), town varchar2(50), vpp number(1),
constraint airport_pk primary key (iata));

create table ada.flight (fn varchar2(10), price number(10,2), airline varchar2(5), afrom varchar2(5), ato varchar2(5),
constraint flight_pk primary key (fn),
constraint flight_fk_airline FOREIGN key (airline) REFERENCES ada.airline(code),
constraint flight_fk_afrom FOREIGN key (afrom) REFERENCES ada.airport(iata),
constraint flight_fk_ato FOREIGN key (ato) REFERENCES ada.airport(iata));

create table ada.airline_airport (code varchar2(5), iata varchar2(5),
constraint airline_airport_pk primary key (code, iata),
constraint airline_airport_fk_airline foreign key (code) REFERENCES ada.airline(code),
constraint airline_airport_fk2_airport foreign key (iata) REFERENCES ada.airport(iata));

insert into ada.airline(name, code, base) values ('Aeroflot', 'AFL', 'Moscow');
insert into ada.airline(name, code, base) values ('Rossia', 'SDM', 'Sankt-Petersburg');
insert into ada.airline(name, code, base) values ('Utair', 'UTA', 'Tumen');

insert into ada.airport (name, vpp, iata, town) values ('Sheremetevo', 3, 'SVO', 'Moscow');
insert into ada.airport (name, vpp, iata, town) values ('Vnukovo', 2, 'VKO', 'Moscow');
insert into ada.airport (name, vpp, iata, town) values ('Adler', 2, 'AER', 'Sochi');
insert into ada.airport (name, vpp, iata, town) values ('Istambul Havalimani Camii', 5, 'IST', 'Istambul');
insert into ada.airport (name, vpp, iata, town) values ('Roschino', 2, 'TJM', 'Tumen');
insert into ada.airport (name, vpp, iata, town) values ('SPb', 2, 'SPB', 'Sankt-Petersburg');

insert into ada.airline_airport (code, iata) values ('AFL','SVO');
insert into ada.airline_airport (code, iata) values ('AFL','AER');
insert into ada.airline_airport (code, iata) values ('AFL','SVO');
insert into ada.airline_airport (code, iata) values ('SDM','VKO');
insert into ada.airline_airport (code, iata) values ('SDM','AER');
insert into ada.airline_airport (code, iata) values ('SDM','TJM');
insert into ada.airline_airport (code, iata) values ('UTA','VKO');
insert into ada.airline_airport (code, iata) values ('UTA','TJM');

insert into ada.flight(fn, airline, price, afrom, ato) values ('SU01', 'AFL', 8500, 'SVO','AER');
insert into ada.flight(fn, airline, price, afrom, ato) values ('SU02', 'AFL', 22000, 'SVO','IST');
insert into ada.flight(fn, airline, price, afrom, ato) values ('SU03', 'AFL', 20000, 'TJM','SVO');
insert into ada.flight(fn, airline, price, afrom, ato) values ('SU04', 'AFL', 15000, 'AER','IST');
insert into ada.flight(fn, airline, price, afrom, ato) values ('R201', 'SDM', 12000, 'TJM','VKO');
insert into ada.flight(fn, airline, price, afrom, ato) values ('UT01', 'UTA', 11000, 'TJM','VKO');
insert into ada.flight(fn, airline, price, afrom, ato) values ('UT02', 'UTA', 10000, 'VKO','AER');
insert into ada.flight(fn, airline, price, afrom, ato) values ('R202', 'SDM', 18000, 'VKO','IST');
insert into ada.flight(fn, airline, price, afrom, ato) values ('SU05', 'AFL', 4500, 'SVO','SPB');
insert into ada.flight(fn, airline, price, afrom, ato) values ('SU06', 'AFL', 9500, 'SPB','AER');
```
