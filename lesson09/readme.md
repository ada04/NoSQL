# Урок 9 - Clickhouse

# Домашнее задание
## Цель
В результате выполнения ДЗ в разверните БД.

## Описание/Пошаговая инструкция выполнения домашнего задания:
Необходимо, используя туториал https://clickhouse.tech/docs/ru/getting-started/tutorial/ :

- развернуть БД;
- выполнить импорт тестовой БД;
- выполнить несколько запросов и оценить скорость выполнения.
- развернуть дополнительно одну из тестовых БД https://clickhouse.com/docs/en/getting-started/example-datasets , протестировать скорость запросов
- развернуть Кликхаус в кластерном исполнении, создать распределенную таблицу, заполнить данными и протестировать скорость по сравнению с 1 инстансом

### Устанавливаем сервер

Первую часть задания будем выполнять на ВМ (VirtualBox) с установленной Ubuntu 22.04.3

Следуя инструкциям устанавливаем Clickhouse:

```bash
grep -q sse4_2 /proc/cpuinfo && echo "SSE 4.2 supported" || echo "SSE 4.2 not supported"

sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates dirmngr

sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754
echo "deb https://packages.clickhouse.com/deb stable main" | sudo tee \
    /etc/apt/sources.list.d/clickhouse.list

sudo apt-get update

sudo apt-get install -y clickhouse-server clickhouse-client
```

В конце установки можно посмотреть что было создано и где размещаются логи, данные и бинарники... (Полный лог установки: [Log](./out_01.log))

```bash
Creating clickhouse group if it does not exist.
 groupadd -r clickhouse
Creating clickhouse user if it does not exist.
 useradd -r --shell /bin/false --home-dir /nonexistent -g clickhouse clickhouse
Will set ulimits for clickhouse user in /etc/security/limits.d/clickhouse.conf.
Creating config directory /etc/clickhouse-server/config.d that is used for tweaks of main server configuration.
Creating config directory /etc/clickhouse-server/users.d that is used for tweaks of users configuration.
Config file /etc/clickhouse-server/config.xml already exists, will keep it and extract path info from it.
/etc/clickhouse-server/config.xml has /var/lib/clickhouse/ as data path.
/etc/clickhouse-server/config.xml has /var/log/clickhouse-server/ as log path.
Users config file /etc/clickhouse-server/users.xml already exists, will keep it and extract users info from it.
Creating log directory /var/log/clickhouse-server/.
Creating data directory /var/lib/clickhouse/.
Creating pid directory /var/run/clickhouse-server.
 chown -R clickhouse:clickhouse '/var/log/clickhouse-server/'
 chown -R clickhouse:clickhouse '/var/run/clickhouse-server'
 chown  clickhouse:clickhouse '/var/lib/clickhouse/'
 groupadd -r clickhouse-bridge
 useradd -r --shell /bin/false --home-dir /nonexistent -g clickhouse-bridge clickhouse-bridge
 chown -R clickhouse-bridge:clickhouse-bridge '/usr/bin/clickhouse-odbc-bridge'
 chown -R clickhouse-bridge:clickhouse-bridge '/usr/bin/clickhouse-library-bridge'
Enter password for default user:
Password for default user is saved in file /etc/clickhouse-server/users.d/default-password.xml.
Setting capabilities for clickhouse binary. This is optional.
 chown -R clickhouse:clickhouse '/etc/clickhouse-server'

ClickHouse has been successfully installed.

Start clickhouse-server with:
 sudo clickhouse start

Start clickhouse-client with:
 clickhouse-client --password
```

Запустим клиент и проверим все ли работает, выполнив зварос **SELECT 1**

```bash
service clickhouse-server start
clickhouse-client
ClickHouse client version 23.12.2.59 (official build).
Connecting to localhost:9000 as user default.
Password for user (default):
Connecting to localhost:9000 as user default.
Connected to ClickHouse server version 23.12.2.

Warnings:
 * Delay accounting is not enabled, OSIOWaitMicroseconds will not be gathered. Check /proc/sys/kernel/task_delayacct
 * Maximum number of threads is lower than 30000. There could be problems with handling a lot of simultaneous queries.

ubuntu2204.localdomain :) SELECT 1

SELECT 1

Query id: 5ee5fd3d-fe78-49f5-a4a9-a92552e815a6

┌─1─┐
│ 1 │
└───┘

1 row in set. Elapsed: 0.002 sec.

ubuntu2204.localdomain :) create database test1 comment 'Database for testa OTUS'

CREATE DATABASE test1
COMMENT 'Database for testa OTUS'

Query id: 2fdbefeb-8fef-4cd8-a388-7c5201a83ee7

Ok.

0 rows in set. Elapsed: 0.042 sec.

ubuntu2204.localdomain :) SELECT name, comment FROM system.databases;

SELECT
    name,
    comment
FROM system.databases

Query id: 51e29299-0cb0-41ba-bce0-e2504fe348ac

┌─name───────────────┬─comment─────────────────┐
│ INFORMATION_SCHEMA │                         │
│ default            │                         │
│ information_schema │                         │
│ system             │                         │
│ test1              │ Database for testa OTUS │
└────────────────────┴─────────────────────────┘

5 rows in set. Elapsed: 0.002 sec.

ubuntu2204.localdomain :) exit
Bye.
root@ubuntu2204:~#

```

Как видим, все запустилось успешно.

### Импортируем тестовую БД

#### Загружаем тестовый набор данных:

```bash

curl https://datasets.clickhouse.com/hits/tsv/hits_v1.tsv.xz | unxz --threads=`nproc` > hits_v1.tsv
curl https://datasets.clickhouse.com/visits/tsv/visits_v1.tsv.xz | unxz --threads=`nproc` > visits_v1.tsv
```

#### Создание таблиц

Таблицы будем создавать в БД **test1**
Оператор CREATE TABLE должен указывать на три ключевых момента:
- Имя создаваемой таблицы.
- Схему таблицы, то есть задавать список столбцов и их типы данных.
- Движок таблицы и его параметры, которые определяют все детали того, как запросы к данной таблице будут физически исполняться.

Мы создадим две таблицы:

- таблицу hits с действиями, осуществлёнными всеми пользователями на всех сайтах, обслуживаемых сервисом;
- таблицу visits, содержащую посещения — преднастроенные сессии вместо каждого действия.

Выполним операторы CREATE TABLE для создания этих таблиц:

```sql
CREATE TABLE test1.hits_v1
(
    `WatchID` UInt64,    `JavaEnable` UInt8,    `Title` String,    `GoodEvent` Int16,    `EventTime` DateTime,    `EventDate` Date,    `CounterID` UInt32,
    `ClientIP` UInt32,    `ClientIP6` FixedString(16),    `RegionID` UInt32,    `UserID` UInt64,    `CounterClass` Int8,    `OS` UInt8,    `UserAgent` UInt8,
    `URL` String,    `Referer` String,    `URLDomain` String,    `RefererDomain` String,    `Refresh` UInt8,    `IsRobot` UInt8,    `RefererCategories` Array(UInt16),
    `URLCategories` Array(UInt16),    `URLRegions` Array(UInt32),    `RefererRegions` Array(UInt32),    `ResolutionWidth` UInt16,    `ResolutionHeight` UInt16,
    `ResolutionDepth` UInt8,    `FlashMajor` UInt8,    `FlashMinor` UInt8,    `FlashMinor2` String,    `NetMajor` UInt8,    `NetMinor` UInt8,    `UserAgentMajor` UInt16,
    `UserAgentMinor` FixedString(2),    `CookieEnable` UInt8,    `JavascriptEnable` UInt8,    `IsMobile` UInt8,    `MobilePhone` UInt8,    `MobilePhoneModel` String,
    `Params` String,    `IPNetworkID` UInt32,    `TraficSourceID` Int8,    `SearchEngineID` UInt16,    `SearchPhrase` String,    `AdvEngineID` UInt8,
    `IsArtifical` UInt8,    `WindowClientWidth` UInt16,    `WindowClientHeight` UInt16,    `ClientTimeZone` Int16,    `ClientEventTime` DateTime,    `SilverlightVersion1` UInt8,
    `SilverlightVersion2` UInt8,    `SilverlightVersion3` UInt32,    `SilverlightVersion4` UInt16,    `PageCharset` String,    `CodeVersion` UInt32,    `IsLink` UInt8,
    `IsDownload` UInt8,    `IsNotBounce` UInt8,    `FUniqID` UInt64,    `HID` UInt32,    `IsOldCounter` UInt8,    `IsEvent` UInt8,    `IsParameter` UInt8,    `DontCountHits` UInt8,
    `WithHash` UInt8,    `HitColor` FixedString(1),    `UTCEventTime` DateTime,    `Age` UInt8,    `Sex` UInt8,    `Income` UInt8,    `Interests` UInt16,    `Robotness` UInt8,
    `GeneralInterests` Array(UInt16),    `RemoteIP` UInt32,    `RemoteIP6` FixedString(16),    `WindowName` Int32,    `OpenerName` Int32,    `HistoryLength` Int16,
    `BrowserLanguage` FixedString(2),    `BrowserCountry` FixedString(2),    `SocialNetwork` String,    `SocialAction` String,    `HTTPError` UInt16,    `SendTiming` Int32,
    `DNSTiming` Int32,    `ConnectTiming` Int32,    `ResponseStartTiming` Int32,    `ResponseEndTiming` Int32,    `FetchTiming` Int32,    `RedirectTiming` Int32,
    `DOMInteractiveTiming` Int32,    `DOMContentLoadedTiming` Int32,    `DOMCompleteTiming` Int32,    `LoadEventStartTiming` Int32,    `LoadEventEndTiming` Int32,
    `NSToDOMContentLoadedTiming` Int32,    `FirstPaintTiming` Int32,    `RedirectCount` Int8,    `SocialSourceNetworkID` UInt8,    `SocialSourcePage` String,    `ParamPrice` Int64,
    `ParamOrderID` String,    `ParamCurrency` FixedString(3),    `ParamCurrencyID` UInt16,    `GoalsReached` Array(UInt32),    `OpenstatServiceName` String,
    `OpenstatCampaignID` String,    `OpenstatAdID` String,    `OpenstatSourceID` String,    `UTMSource` String,    `UTMMedium` String,    `UTMCampaign` String,    `UTMContent` String,
    `UTMTerm` String,    `FromTag` String,    `HasGCLID` UInt8,    `RefererHash` UInt64,    `URLHash` UInt64,    `CLID` UInt32,    `YCLID` UInt64,    `ShareService` String,
    `ShareURL` String,    `ShareTitle` String,    `ParsedParams` Nested(Key1 String, Key2 String, Key3 String, Key4 String, Key5 String,  ValueDouble Float64),
    `IslandID` FixedString(16),    `RequestNum` UInt32,    `RequestTry` UInt8
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID))
SAMPLE BY intHash32(UserID)
```

```sql
CREATE TABLE test1.visits_v1
(    `CounterID` UInt32,
    `StartDate` Date,
    `Sign` Int8,
    `IsNew` UInt8,
    `VisitID` UInt64,
    `UserID` UInt64,
    `StartTime` DateTime,
    `Duration` UInt32,
    `UTCStartTime` DateTime,
    `PageViews` Int32,
    `Hits` Int32,
    `IsBounce` UInt8,
    `Referer` String,
    `StartURL` String,
    `RefererDomain` String,
    `StartURLDomain` String,
    `EndURL` String,
    `LinkURL` String,
    `IsDownload` UInt8,
    `TraficSourceID` Int8,
    `SearchEngineID` UInt16,
    `SearchPhrase` String,
    `AdvEngineID` UInt8,
    `PlaceID` Int32,
    `RefererCategories` Array(UInt16),
    `URLCategories` Array(UInt16),
    `URLRegions` Array(UInt32),
    `RefererRegions` Array(UInt32),
    `IsYandex` UInt8,
    `GoalReachesDepth` Int32,
    `GoalReachesURL` Int32,
    `GoalReachesAny` Int32,
    `SocialSourceNetworkID` UInt8,
    `SocialSourcePage` String,
    `MobilePhoneModel` String,
    `ClientEventTime` DateTime,
    `RegionID` UInt32,
    `ClientIP` UInt32,
    `ClientIP6` FixedString(16),
    `RemoteIP` UInt32,
    `RemoteIP6` FixedString(16),
    `IPNetworkID` UInt32,
    `SilverlightVersion3` UInt32,
    `CodeVersion` UInt32,
    `ResolutionWidth` UInt16,
    `ResolutionHeight` UInt16,
    `UserAgentMajor` UInt16,
    `UserAgentMinor` UInt16,
    `WindowClientWidth` UInt16,
    `WindowClientHeight` UInt16,
    `SilverlightVersion2` UInt8,
    `SilverlightVersion4` UInt16,
    `FlashVersion3` UInt16,
    `FlashVersion4` UInt16,
    `ClientTimeZone` Int16,
    `OS` UInt8,
    `UserAgent` UInt8,
    `ResolutionDepth` UInt8,
    `FlashMajor` UInt8,
    `FlashMinor` UInt8,
    `NetMajor` UInt8,
    `NetMinor` UInt8,
    `MobilePhone` UInt8,
    `SilverlightVersion1` UInt8,
    `Age` UInt8,
    `Sex` UInt8,
    `Income` UInt8,
    `JavaEnable` UInt8,
    `CookieEnable` UInt8,
    `JavascriptEnable` UInt8,
    `IsMobile` UInt8,
    `BrowserLanguage` UInt16,
    `BrowserCountry` UInt16,
    `Interests` UInt16,
    `Robotness` UInt8,
    `GeneralInterests` Array(UInt16),
    `Params` Array(String),
    `Goals` Nested(
        ID UInt32,
        Serial UInt32,
        EventTime DateTime,
        Price Int64,
        OrderID String,
        CurrencyID UInt32),
    `WatchIDs` Array(UInt64),
    `ParamSumPrice` Int64,
    `ParamCurrency` FixedString(3),
    `ParamCurrencyID` UInt16,
    `ClickLogID` UInt64,
    `ClickEventID` Int32,
    `ClickGoodEvent` Int32,
    `ClickEventTime` DateTime,
    `ClickPriorityID` Int32,
    `ClickPhraseID` Int32,
    `ClickPageID` Int32,
    `ClickPlaceID` Int32,
    `ClickTypeID` Int32,
    `ClickResourceID` Int32,
    `ClickCost` UInt32,
    `ClickClientIP` UInt32,
    `ClickDomainID` UInt32,
    `ClickURL` String,
    `ClickAttempt` UInt8,
    `ClickOrderID` UInt32,
    `ClickBannerID` UInt32,
    `ClickMarketCategoryID` UInt32,
    `ClickMarketPP` UInt32,
    `ClickMarketCategoryName` String,
    `ClickMarketPPName` String,
    `ClickAWAPSCampaignName` String,
    `ClickPageName` String,
    `ClickTargetType` UInt16,
    `ClickTargetPhraseID` UInt64,
    `ClickContextType` UInt8,
    `ClickSelectType` Int8,
    `ClickOptions` String,
    `ClickGroupBannerID` Int32,
    `OpenstatServiceName` String,
    `OpenstatCampaignID` String,
    `OpenstatAdID` String,
    `OpenstatSourceID` String,
    `UTMSource` String,
    `UTMMedium` String,
    `UTMCampaign` String,
    `UTMContent` String,
    `UTMTerm` String,
    `FromTag` String,
    `HasGCLID` UInt8,
    `FirstVisit` DateTime,
    `PredLastVisit` Date,
    `LastVisit` Date,
    `TotalVisits` UInt32,
    `TraficSource` Nested(
        ID Int8,
        SearchEngineID UInt16,
        AdvEngineID UInt8,
        PlaceID UInt16,
        SocialSourceNetworkID UInt8,
        Domain String,
        SearchPhrase String,
        SocialSourcePage String),
    `Attendance` FixedString(16),
    `CLID` UInt32,
    `YCLID` UInt64,
    `NormalizedRefererHash` UInt64,
    `SearchPhraseHash` UInt64,
    `RefererDomainHash` UInt64,
    `NormalizedStartURLHash` UInt64,
    `StartURLDomainHash` UInt64,
    `NormalizedEndURLHash` UInt64,
    `TopLevelDomain` UInt64,
    `URLScheme` UInt64,
    `OpenstatServiceNameHash` UInt64,
    `OpenstatCampaignIDHash` UInt64,
    `OpenstatAdIDHash` UInt64,
    `OpenstatSourceIDHash` UInt64,
    `UTMSourceHash` UInt64,
    `UTMMediumHash` UInt64,
    `UTMCampaignHash` UInt64,
    `UTMContentHash` UInt64,
    `UTMTermHash` UInt64,
    `FromHash` UInt64,
    `WebVisorEnabled` UInt8,
    `WebVisorActivity` UInt32,
    `ParsedParams` Nested(
        Key1 String,
        Key2 String,
        Key3 String,
        Key4 String,
        Key5 String,
        ValueDouble Float64),
    `Market` Nested(
        Type UInt8,
        GoalID UInt32,
        OrderID String,
        OrderPrice Int64,
        PP UInt32,
        DirectPlaceID UInt32,
        DirectOrderID UInt32,
        DirectBannerID UInt32,
        GoodID String,
        GoodName String,
        GoodQuantity Int32,
        GoodPrice Int64),
    `IslandID` FixedString(16)
)
ENGINE = CollapsingMergeTree(Sign)
PARTITION BY toYYYYMM(StartDate)
ORDER BY (CounterID, StartDate, intHash32(UserID), VisitID)
SAMPLE BY intHash32(UserID)
```

Таблица hits_v1 использует базовый вариант движка MergeTree, а visits_v1 использует вариант Collapsing.

#### Импорт данных

Импорт данных в ClickHouse выполняется оператором INSERT INTO как в большинстве SQL-систем. Однако данные для вставки в таблицы ClickHouse обычно предоставляются в одном из поддерживаемых форматов вместо их непосредственного указания в предложении VALUES (хотя и этот способ поддерживается).

У нас загружены файлы в формате со значениями, разделёнными знаком табуляции; импортируем их, указав соответствующие запросы в аргументах командной строки:

```bash
clickhouse-client --query "INSERT INTO test1.hits_v1 FORMAT TSV" --max_insert_block_size=100000 < hits_v1.tsv --password aa2024
clickhouse-client --query "INSERT INTO test1.visits_v1 FORMAT TSV" --max_insert_block_size=100000 < visits_v1.tsv  --password aa2024
```

Посмотреть какие настройки доступны, что они означают и какие у них значения по умолчанию можно в таблице system.settings:

```sql
SELECT name, value, changed, description
FROM system.settings
WHERE name LIKE '%max_insert_b%'
FORMAT TSV
```

После импорта применим к таблицам оператор OPTIMIZE. Этот оператор принудительно запускает соответствующие процессы слияния вместо того, чтобы эти действия были выполнены в фоне когда-нибудь позже.

```sql
OPTIMIZE TABLE test1.hits_v1 FINAL
OPTIMIZE TABLE test1.visits_v1 FINAL
```

Проверим, успешно ли загрузились данные:

```sql
SELECT COUNT(*) FROM test1.hits_v1
SELECT COUNT(*) FROM test1.visits_v1
```

```
ubuntu2204.localdomain :) SELECT COUNT(*) FROM test1.hits_v1

SELECT COUNT(*)
FROM test1.hits_v1

Query id: 9740aca5-c3f7-4d99-8156-b80919d4dde1

┌─count()─┐
│ 8873898 │
└─────────┘

1 row in set. Elapsed: 0.046 sec.

ubuntu2204.localdomain :) SELECT COUNT(*) FROM test1.visits_v1

SELECT COUNT(*)
FROM test1.visits_v1

Query id: 1ad4e149-c4b0-4bb8-8728-8d84cb5dfedd

┌─count()─┐
│ 1676861 │
└─────────┘

1 row in set. Elapsed: 0.004 sec.

ubuntu2204.localdomain :)
```

Примеры запросов

```sql
SELECT
    StartURL AS URL,
    AVG(Duration) AS AvgDuration
FROM test1.visits_v1
WHERE StartDate BETWEEN '2014-03-20' AND '2014-03-22'
GROUP BY URL
ORDER BY AvgDuration DESC
LIMIT 10

SELECT
    sum(Sign) AS visits,
    sumIf(Sign, has(Goals.ID, 1105530)) AS goal_visits,
    (100. * goal_visits) / visits AS goal_percent
FROM test1.visits_v1
WHERE (CounterID = 912887) AND (toYYYYMM(StartDate) = 201403)
```

Несмотря на большое количество записей и работу ClickHouse в ВМ на обычном ПК, время выполнения запросов составило 0.271 и 0.041 сек.

