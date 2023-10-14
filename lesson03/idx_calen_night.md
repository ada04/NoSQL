**Сравнение выполнения запросов с индексом и без по полю `minimum_night` (индекс по полю `date` создан ранее)**

Запрос

    db.calendar.find({ $and:  [{date:{ $gt: '2023-10-15'}}, {minimum_nights: 1}]}).explain("executionStats")
    
Создание индекса

    db.calendar.createIndex({ minimum_nights: 1 })
    
Без индекса

```
mydb> db.calendar.find({ $and:  [{date:{ $gt: '2023-10-15'}}, {minimum_nights: 1}]}).explain("executionStats")
{
  explainVersion: '2',
  queryPlanner: {
    namespace: 'mydb.calendar',
    indexFilterSet: false,
    parsedQuery: {
      '$and': [
        { minimum_nights: { '$eq': 1 } },
        { date: { '$gt': '2023-10-15' } }
      ]
    },
    queryHash: '5EBB95CC',
    planCacheKey: 'A662B947',
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    winningPlan: {
      queryPlan: {
        stage: 'FETCH',
        planNodeId: 2,
        filter: { minimum_nights: { '$eq': 1 } },
        inputStage: {
          stage: 'IXSCAN',
          planNodeId: 1,
          keyPattern: { date: 1 },
          indexName: 'date_1',
          isMultiKey: false,
          multiKeyPaths: { date: [] },
          isUnique: false,
          isSparse: false,
          isPartial: false,
          indexVersion: 2,
          direction: 'forward',
          indexBounds: { date: [ '("2023-10-15", {})' ] }
        }
      },
      slotBasedPlan: {
        slots: '$$RESULT=s11 env: { s1 = TimeZoneDatabase(America/Noronha...America/Danmarkshavn) (timeZoneDB), s6 = KS(46000104), s10 = {"date" : 1}, s14 = 1, s5 = KS(3C323032332D31302D313500FE04), s2 = Nothing (SEARCH_META), s3 = 1697262015282 (NOW) }',
        stages: '[2] filter {traverseF(s13, lambda(l1.0) { ((l1.0 == s14) ?: false) }, false)} \n' +
          '[2] nlj inner [] [s4, s7, s8, s9, s10] \n' +
          '    left \n' +
          '        [1] cfilter {(exists(s5) && exists(s6))} \n' +
          '        [1] ixseek s5 s6 s9 s4 s7 s8 [] @"f608ea24-04f3-4d10-ab13-ea4936e82c7e" @"date_1" true \n' +
          '    right \n' +
          '        [2] limit 1 \n' +
          '        [2] seek s4 s11 s12 s7 s8 s9 s10 [s13 = minimum_nights] @"f608ea24-04f3-4d10-ab13-ea4936e82c7e" true false \n'
      }
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 70727,
    executionTimeMillis: 884,
    totalKeysExamined: 360468,
    totalDocsExamined: 360468,
    executionStages: {
      stage: 'filter',
      planNodeId: 2,
      nReturned: 70727,
      executionTimeMillisEstimate: 881,
      opens: 1,
      closes: 1,
      saveState: 360,
      restoreState: 360,
      isEOF: 1,
      numTested: 360468,
      filter: 'traverseF(s13, lambda(l1.0) { ((l1.0 == s14) ?: false) }, false) ',
      inputStage: {
        stage: 'nlj',
        planNodeId: 2,
        nReturned: 360468,
        executionTimeMillisEstimate: 873,
        opens: 1,
        closes: 1,
        saveState: 360,
        restoreState: 360,
        isEOF: 1,
        totalDocsExamined: 360468,
        totalKeysExamined: 360468,
        collectionScans: 0,
        collectionSeeks: 360468,
        indexScans: 0,
        indexSeeks: 1,
        indexesUsed: [ 'date_1' ],
        innerOpens: 360468,
        innerCloses: 1,
        outerProjects: [],
        outerCorrelated: [ Long("4"), Long("7"), Long("8"), Long("9"), Long("10") ],
        outerStage: {
          stage: 'cfilter',
          planNodeId: 1,
          nReturned: 360468,
          executionTimeMillisEstimate: 738,
          opens: 1,
          closes: 1,
          saveState: 360,
          restoreState: 360,
          isEOF: 1,
          numTested: 1,
          filter: '(exists(s5) && exists(s6)) ',
          inputStage: {
            stage: 'ixseek',
            planNodeId: 1,
            nReturned: 360468,
            executionTimeMillisEstimate: 738,
            opens: 1,
            closes: 1,
            saveState: 360,
            restoreState: 360,
            isEOF: 1,
            indexName: 'date_1',
            keysExamined: 360468,
            seeks: 1,
            numReads: 360469,
            indexKeySlot: 9,
            recordIdSlot: 4,
            snapshotIdSlot: 7,
            indexIdentSlot: 8,
            outputSlots: [],
            indexKeysToInclude: '00000000000000000000000000000000',
            seekKeyLow: 's5 ',
            seekKeyHigh: 's6 '
          }
        },
        innerStage: {
          stage: 'limit',
          planNodeId: 2,
          nReturned: 360468,
          executionTimeMillisEstimate: 131,
          opens: 360468,
          closes: 1,
          saveState: 360,
          restoreState: 360,
          isEOF: 1,
          limit: 1,
          inputStage: {
            stage: 'seek',
            planNodeId: 2,
            nReturned: 360468,
            executionTimeMillisEstimate: 129,
            opens: 360468,
            closes: 1,
            saveState: 360,
            restoreState: 360,
            isEOF: 0,
            numReads: 360468,
            recordSlot: 11,
            recordIdSlot: 12,
            seekKeySlot: 4,
            snapshotIdSlot: 7,
            indexIdentSlot: 8,
            indexKeySlot: 9,
            indexKeyPatternSlot: 10,
            fields: [ 'minimum_nights' ],
            outputSlots: [ Long("13") ]
          }
        }
      }
    }
  },
  command: {
    find: 'calendar',
    filter: {
      '$and': [ { date: { '$gt': '2023-10-15' } }, { minimum_nights: 1 } ]
    },
    '$db': 'mydb'
  },
  serverInfo: {
    host: 'srv-mongodb-01',
    port: 27017,
    version: '7.0.2',
    gitVersion: '02b3c655e1302209ef046da6ba3ef6749dd0b62a'
  },
  serverParameters: {
    internalQueryFacetBufferSizeBytes: 104857600,
    internalQueryFacetMaxOutputDocSizeBytes: 104857600,
    internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
    internalDocumentSourceGroupMaxMemoryBytes: 104857600,
    internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
    internalQueryProhibitBlockingMergeOnMongoS: 0,
    internalQueryMaxAddToSetBytes: 104857600,
    internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600,
    internalQueryFrameworkControl: 'trySbeEngine'
  },
  ok: 1
}
```

С индексом

```
mydb> db.calendar.createIndex({ minimum_nights: 1 })
minimum_nights_1
mydb> db.calendar.find({ $and:  [{date:{ $gt: '2023-10-15'}}, {minimum_nights: 1}]}).explain("executionStats")
{
  explainVersion: '2',
  queryPlanner: {
    namespace: 'mydb.calendar',
    indexFilterSet: false,
    parsedQuery: {
      '$and': [
        { minimum_nights: { '$eq': 1 } },
        { date: { '$gt': '2023-10-15' } }
      ]
    },
    queryHash: '5EBB95CC',
    planCacheKey: '77F92B03',
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    winningPlan: {
      queryPlan: {
        stage: 'FETCH',
        planNodeId: 2,
        filter: { date: { '$gt': '2023-10-15' } },
        inputStage: {
          stage: 'IXSCAN',
          planNodeId: 1,
          keyPattern: { minimum_nights: 1 },
          indexName: 'minimum_nights_1',
          isMultiKey: false,
          multiKeyPaths: { minimum_nights: [] },
          isUnique: false,
          isSparse: false,
          isPartial: false,
          indexVersion: 2,
          direction: 'forward',
          indexBounds: { minimum_nights: [ '[1, 1]' ] }
        }
      },
      slotBasedPlan: {
        slots: '$$RESULT=s11 env: { s1 = TimeZoneDatabase(America/Noronha...America/Danmarkshavn) (timeZoneDB), s6 = KS(2B02FE04), s10 = {"minimum_nights" : 1}, s14 = "2023-10-15", s5 = KS(2B020104), s2 = Nothing (SEARCH_META), s3 = 1697262302281 (NOW) }',
        stages: '[2] filter {traverseF(s13, lambda(l1.0) { ((l1.0 > s14) ?: false) }, false)} \n' +
          '[2] nlj inner [] [s4, s7, s8, s9, s10] \n' +
          '    left \n' +
          '        [1] cfilter {(exists(s5) && exists(s6))} \n' +
          '        [1] ixseek s5 s6 s9 s4 s7 s8 [] @"f608ea24-04f3-4d10-ab13-ea4936e82c7e" @"minimum_nights_1" true \n' +
          '    right \n' +
          '        [2] limit 1 \n' +
          '        [2] seek s4 s11 s12 s7 s8 s9 s10 [s13 = date] @"f608ea24-04f3-4d10-ab13-ea4936e82c7e" true false \n'
      }
    },
    rejectedPlans: [
      {
        queryPlan: {
          stage: 'FETCH',
          planNodeId: 2,
          filter: { minimum_nights: { '$eq': 1 } },
          inputStage: {
            stage: 'IXSCAN',
            planNodeId: 1,
            keyPattern: { date: 1 },
            indexName: 'date_1',
            isMultiKey: false,
            multiKeyPaths: { date: [] },
            isUnique: false,
            isSparse: false,
            isPartial: false,
            indexVersion: 2,
            direction: 'forward',
            indexBounds: { date: [ '("2023-10-15", {})' ] }
          }
        },
        slotBasedPlan: {
          slots: '$$RESULT=s11 env: { s1 = TimeZoneDatabase(America/Noronha...America/Danmarkshavn) (timeZoneDB), s6 = KS(46000104), s10 = {"date" : 1}, s14 = 1, s5 = KS(3C323032332D31302D313500FE04), s2 = Nothing (SEARCH_META), s3 = 1697262302281 (NOW) }',
          stages: '[2] filter {traverseF(s13, lambda(l1.0) { ((l1.0 == s14) ?: false) }, false)} \n' +
            '[2] nlj inner [] [s4, s7, s8, s9, s10] \n' +
            '    left \n' +
            '        [1] cfilter {(exists(s5) && exists(s6))} \n' +
            '        [1] ixseek s5 s6 s9 s4 s7 s8 [] @"f608ea24-04f3-4d10-ab13-ea4936e82c7e" @"date_1" true \n' +
            '    right \n' +
            '        [2] limit 1 \n' +
            '        [2] seek s4 s11 s12 s7 s8 s9 s10 [s13 = minimum_nights] @"f608ea24-04f3-4d10-ab13-ea4936e82c7e" true false \n'
        }
      }
    ]
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 70727,
    executionTimeMillis: 189,
    totalKeysExamined: 75552,
    totalDocsExamined: 75552,
    executionStages: {
      stage: 'filter',
      planNodeId: 2,
      nReturned: 70727,
      executionTimeMillisEstimate: 177,
      opens: 1,
      closes: 1,
      saveState: 76,
      restoreState: 76,
      isEOF: 1,
      numTested: 75552,
      filter: 'traverseF(s13, lambda(l1.0) { ((l1.0 > s14) ?: false) }, false) ',
      inputStage: {
        stage: 'nlj',
        planNodeId: 2,
        nReturned: 75552,
        executionTimeMillisEstimate: 174,
        opens: 1,
        closes: 1,
        saveState: 76,
        restoreState: 76,
        isEOF: 1,
        totalDocsExamined: 75552,
        totalKeysExamined: 75552,
        collectionScans: 0,
        collectionSeeks: 75552,
        indexScans: 0,
        indexSeeks: 1,
        indexesUsed: [ 'minimum_nights_1' ],
        innerOpens: 75552,
        innerCloses: 1,
        outerProjects: [],
        outerCorrelated: [ Long("4"), Long("7"), Long("8"), Long("9"), Long("10") ],
        outerStage: {
          stage: 'cfilter',
          planNodeId: 1,
          nReturned: 75552,
          executionTimeMillisEstimate: 147,
          opens: 1,
          closes: 1,
          saveState: 76,
          restoreState: 76,
          isEOF: 1,
          numTested: 1,
          filter: '(exists(s5) && exists(s6)) ',
          inputStage: {
            stage: 'ixseek',
            planNodeId: 1,
            nReturned: 75552,
            executionTimeMillisEstimate: 147,
            opens: 1,
            closes: 1,
            saveState: 76,
            restoreState: 76,
            isEOF: 1,
            indexName: 'minimum_nights_1',
            keysExamined: 75552,
            seeks: 1,
            numReads: 75553,
            indexKeySlot: 9,
            recordIdSlot: 4,
            snapshotIdSlot: 7,
            indexIdentSlot: 8,
            outputSlots: [],
            indexKeysToInclude: '00000000000000000000000000000000',
            seekKeyLow: 's5 ',
            seekKeyHigh: 's6 '
          }
        },
        innerStage: {
          stage: 'limit',
          planNodeId: 2,
          nReturned: 75552,
          executionTimeMillisEstimate: 26,
          opens: 75552,
          closes: 1,
          saveState: 76,
          restoreState: 76,
          isEOF: 1,
          limit: 1,
          inputStage: {
            stage: 'seek',
            planNodeId: 2,
            nReturned: 75552,
            executionTimeMillisEstimate: 17,
            opens: 75552,
            closes: 1,
            saveState: 76,
            restoreState: 76,
            isEOF: 0,
            numReads: 75552,
            recordSlot: 11,
            recordIdSlot: 12,
            seekKeySlot: 4,
            snapshotIdSlot: 7,
            indexIdentSlot: 8,
            indexKeySlot: 9,
            indexKeyPatternSlot: 10,
            fields: [ 'date' ],
            outputSlots: [ Long("13") ]
          }
        }
      }
    }
  },
  command: {
    find: 'calendar',
    filter: {
      '$and': [ { date: { '$gt': '2023-10-15' } }, { minimum_nights: 1 } ]
    },
    '$db': 'mydb'
  },
  serverInfo: {
    host: 'srv-mongodb-01',
    port: 27017,
    version: '7.0.2',
    gitVersion: '02b3c655e1302209ef046da6ba3ef6749dd0b62a'
  },
  serverParameters: {
    internalQueryFacetBufferSizeBytes: 104857600,
    internalQueryFacetMaxOutputDocSizeBytes: 104857600,
    internalLookupStageIntermediateDocumentMaxSizeBytes: 104857600,
    internalDocumentSourceGroupMaxMemoryBytes: 104857600,
    internalQueryMaxBlockingSortMemoryUsageBytes: 104857600,
    internalQueryProhibitBlockingMergeOnMongoS: 0,
    internalQueryMaxAddToSetBytes: 104857600,
    internalDocumentSourceSetWindowFieldsMaxMemoryBytes: 104857600,
    internalQueryFrameworkControl: 'trySbeEngine'
  },
  ok: 1
}
mydb>
```
