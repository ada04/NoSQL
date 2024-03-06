# Урок 21 Neo4j, часть 2

## Теория:
[Запросы](neo4j_p1.md)

[movie.txt](movie.txt)


# Домашнее задание

Использование neo4j

## Цель:
- Научиться представлять данные в neo4j
- Научиться использовать Cypher для манипуляции данными в neo4j
- Научиться смотреть план запроса
- Создание индексов

## Описание/Пошаговая инструкция выполнения домашнего задания:
- Взять 4-5 популярных туроператора.
- Каждый туроператор должен быть представлен в виде ноды neo4j
- Взять 10-15 направлений, в которые данные операторы предосавляют путевки.
- Представить направления в виде связки нод: страна - конкретное место
- Взять ближайшие к туриситческим локацимя города, в которых есть аэропорты или вокзалы и представить их в виде нод
- Представить маршруты между городми в виде связей. Каждый маршрут должен быть охарактеризован видом транспорта, который позволяет переместиться между точками.
- Написать запрос, который бы выводил направление (со всеми промежуточными точками), который можно осуществить только наземным транспортом.
- Составить план запроса из пункта 7.
- Добавить индексы для оптимизации запроса
- Еще раз посмотреть план запроса и убедиться, что индексыпозволили оптимизировать запрос

### Создание нод и связей

```cypher
create (tt:company {name: 'TezTour', site: 'https://www.tez-tour.travel/'})
create (int:company {name: 'Intourist', site: 'https://intourist.ru/'})
create (peg:company {name: 'Pegas',site: 'https://pegastt.ru/'})
create (cor:company {name: 'Coral', site: 'https://www.coral.ru/'})
create (msk:town {name:'Moscow'})
create (ant:town {name:'Antaliya'})
create (gud1:town {name:'Gudauri'})
create (shar:town {name:'Sharm-El-Sheikh'})
create (hur:town {name:'Hurgada'})
create (san:town {name:'Sanya'})
create (adl:town {name:'Adler'})
create (derb:town {name:'Derbent'})
create (petr:town {name:'Petrozavodsk'})
create (side:town {name:'Side'})
create (galt:town {name:'Gorno-Altaisk'})
create (h01:hotel {name:'Latanya Palm Hotel', star: 5, pans: 'UAI'})
create (h02:hotel {name:'Hotel Deka', star: 2, pans: 'BB'})
create (h03:hotel {name:'Regency & Lodge Hotel', star: 5, pans: 'AI'})
create (h04:hotel {name:'White Valley Palace', star: 4, pans: 'UAI'})
create (h05:hotel {name:'Grand Metro Park Bay', star: 3, pans: 'BD'})
create (h06:hotel {name:'Bogatyr', star: 4, pans: 'FB'})
create (h07:hotel {name:'Phoenix', star: 2, pans: 'BB'})
create (h08:hotel {name:'Sohi Park Hotel', star: 3, pans: 'N'})
create (h09:hotel {name:'Karelia', star: 0, pans: 'N'})
create (h10:hotel {name:'Barut Himera', star: 5, pans: 'UAI'})
create (h11:hotel {name:'Altay', star: 3, pans: 'FB'})
create (h12:hotel {name:'Sunny Hotel', star: 4, pans: 'AI'})
create (rus:country {name: 'Russia'})
create (tur:country {name: 'Turkey'})
create (geo1:country {name: 'Georgia'})
create (egy:country {name: 'Egypt'})
create (chi:country {name: 'China'})
create (tt) -[:tours]-> (h01)
create (tt) -[:tours]-> (h02)
create (tt) -[:tours]-> (h03)
create (int) -[:tours]-> (h04)
create (int) -[:tours]-> (h05)
create (int) -[:tours]-> (h06)
create (peg) -[:tours]-> (h07)
create (peg) -[:tours]-> (h08)
create (peg) -[:tours]-> (h09)
create (cor) -[:tours]-> (h10)
create (cor) -[:tours]-> (h11)
create (cor) -[:tours]-> (h12)
create (tur) -[:dir]-> (ant)
create (geo1) -[:dir]-> (gud1)
create (egy) -[:dir]-> (shar)
create (egy) -[:dir]-> (hur)
create (chi) -[:dir]-> (san)
create (rus) -[:dir]-> (adl)
create (rus) -[:dir]-> (derb)
create (rus) -[:dir]-> (petr)
create (rus) -[:dir]-> (galt)
create (tur) -[:dir]-> (side)
create (rus) -[:dir]-> (msk)
create (ant) -[:location]-> (h01)
create (gud1) -[:location]-> (h02)
create (shar) -[:location]-> (h03)
create (hur) -[:location]-> (h04)
create (san) -[:location]-> (h05)
create (adl) -[:location]-> (h06)
create (derb) -[:location]-> (h07)
create (adl) -[:location]-> (h08)
create (petr) -[:location]-> (h09)
create (side) -[:location]-> (h10)
create (galt) -[:location]-> (h11)
create (msk) -[:location]-> (h12)
create (msk) -[:ton {type: 'land', by: 'auto'}]-> (gud1)
create (msk) -[:ton {type: 'air', by: 'aero'}]-> (shar)
create (msk) -[:ton {type: 'air', by: 'aero'}]-> (hur)
create (msk) -[:ton {type: 'air', by: 'aero'}]-> (san)
create (msk) -[:ton {type: 'air', by: 'aero'}]-> (adl)
create (msk) -[:ton {type: 'land', by: 'rail'}]-> (adl)
create (msk) -[:ton {type: 'air', by: 'aero'}]-> (derb)
create (msk) -[:ton {type: 'land', by: 'rail'}]-> (derb)
create (msk) -[:ton {type: 'land', by: 'rail'}]-> (petr)
create (msk) -[:ton {type: 'land', by: 'auto'}]-> (petr)
create (msk) -[:ton {type: 'air', by: 'aero'}]-> (side)
create (msk) -[:ton {type: 'air', by: 'aero'}]-> (galt)
create (msk) -[:ton {type: 'air', by: 'aero'}]-> (ant)
create (msk) -[:ton {type: 'land', by: 'auto'}]-> (msk)
```
Created 32 nodes, created 49 relationships, set 88 properties, added 32 labels


```cypher
```


```cypher
```


```cypher
```


```cypher
```


```cypher
```
