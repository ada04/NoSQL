#1 получить все сущности из БД
match (n) return n

#2 Удалить все сущности в бд
match (n) detach delete n

#3 создание ноды
create (:Director {name:'Joel Coen'})
create (:Movie {title:'Blood Simple', year:1983})

#4 создание связи между существующими нодами, joel и blood - переменные
match (joel:Director {name:'Joel Coen'})
match (blood:Movie {title:'Blood Simple'})
create (joel) -[:CREATED]-> (blood)

#5 создание новой ноды и связи с существующей нодой
match (blood:Movie {title:'Blood Simple'})
create (:Actor {name: 'Frances McDormand'}) -[:PLAYED_IN]-> (blood)

#6 удаляем всё из базы
match (n) detach delete n

#7 создание сразу нескольких нод и связей
create (:Director {name:'Joel Coen'}) -[:CREATED]-> (:Movie {title:'Blood Simple', year:1983}) <-[:PLAYED_IN]- (:Actor {name: 'Frances McDormand'})

#8 пробуем создать ноду 2 раза
create (:Director {name:'Martin Scorsese'})
create (:Director {name:'Martin Scorsese'})

#9 создать ноду, если не существует
merge (:Director {name: 'Ethan Coen'})
merge (:Director {name: 'Ethan Coen'})

#10 создать связь, если не существует
match (n:Director {name: 'Ethan Coen'})
match (m:Movie {title: 'Blood Simple'})
merge (n) -[:CREATED]-> (m)

#11 добавить свойство к ноде
match (n:Director {name:'Ethan Coen'})
SET n.born = 1957

#12 добавить свойство к связи
match (:Actor {name:'Frances McDormand'}) -[r:PLAYED_IN]-> (:Movie {title: 'Blood Simple'})
set r.character = 'Abby'

#13 удалить ноду
match (martin:Director {name:'Martin Scorsese'}) delete martin

#14 удалить свойство с помощью REMOVE
match (n:Director {name:'Ethan Coen'})
REMOVE n.born

#15 удалить свойство с помощью SET null
match (n:Director {name:'Ethan Coen'})
SET n.born = null

#16 удалить все ребра для ноды
match (n:Director {name:'Ethan Coen'}) -[r]- () delete r

#17 найти ноду
match (joel:Director {name: 'Joel Coen'})

#18 найти ноду имеющую связь с другой нодой с указание метки Label нод
match (d:Director) -[r]- (m:Movie) return d, r, m

#19 найти любые ноды, имеющие связь с другими нодами
…

#20 почистим все
match (n) detach delete n

#21 создадим данные
create (:Director {name:'Joel Coen'}) -[:CREATED]-> (blood:Movie {title:'Blood Simple', year:1983}) <-[:PLAYED_IN {character: 'Abby'}]- (:Actor {name: 'Frances McDormand'})
create (:Director {name:'Ethan Coen', born:1957}) -[:CREATED]-> (blood)

#22 создадим еще больше данных
match (frances:Actor {name:'Frances McDormand'})
match (leo:Actor {name:'Leonardo DiCaprio'})
create (:Director {name:'Martin McDonagh'}) -[:CREATED]-> (billboards:Movie {title:'Three Billboards Outside Ebbing, Missouri'})
create (frances) -[:PLAYED_IN]-> (billboards)
create (venom:Movie {title:'Venom'}) <-[:PLAYED_IN]- (woodie:Actor {name:'Woody Harrelson'}) -[:PLAYED_IN]-> (billboards)
create (venom) <-[:PLAYED_IN]- (tom:Actor {name:'Tom Hardy'})
create (leo) -[:PLAYED_IN]-> (inception:Movie {name:'Inception'}) <-[:PLAYED_IN]- (tom)
create (marion:Actor {name:'Marion Cotillard'}) -[:PLAYED_IN]-> (inception)
create (marion) -[:PLAYED_IN]-> (:Movie {title: 'The Dark Knight Rises'}) <-[:PLAYED_IN]- (tom)
create (nolan:Director {name:'Christopher Nolan'}) -[:CREATED]-> (batman)
create (nolan) -[:CREATED]-> (inception)
create (:Director {name:'Ruben Fleischer'}) -[:CREATED]-> (venom)

#23 найти ноду имеющую связь с другой нодой с указание метки Label нод
match (venom:Movie {title:'Venom'}) -[*1..3]- (d:Director) return d
