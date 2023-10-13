**Сравнение выполнения запросов с индексом или без по полю `date`**

Запрос

    db.calendar.find({ $and:  [{date:{ $gt: '2023-10-15'}}, {date:{ $lt: '2023-11-01'}}]}).explain("executionStats")
    
Создание индекса

    db.calendar.createIndex({  date: 1})
    
Без индекса

```
mydb> db.calendar.find({ $and:  [{date:{ $gt: '2023-10-15'}}, {date:{ $lt: '2023-11-01'}}]}).explain("executionStats")
{
  explainVersion: '2',
  queryPlanner: {
    namespace: 'mydb.calendar',
    indexFilterSet: false,
    parsedQuery: {
      '$and': [
        { date: { '$lt': '2023-11-01' } },
        { date: { '$gt': '2023-10-15' } }
      ]
    },
    queryHash: '3D5C42CD',
    planCacheKey: '2A6F5D6D',
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    winningPlan: {
      queryPlan: {
        stage: 'COLLSCAN',
        planNodeId: 1,
        filter: {
          '$and': [
            { date: { '$lt': '2023-11-01' } },
            { date: { '$gt': '2023-10-15' } }
          ]
        },
        direction: 'forward'
      },
      slotBasedPlan: {
        slots: '$$RESULT=s5 env: { s2 = Nothing (SEARCH_META), s8 = "2023-10-15", s3 = 1697219215414 (NOW), s1 = TimeZoneDatabase(America/Noronha...America/Danmarkshavn) (timeZoneDB), s7 = "2023-11-01" }',
        stages: '[1] filter {(traverseF(s4, lambda(l1.0) { ((l1.0 < s7) ?: false) }, false) && traverseF(s4, lambda(l2.0) { ((l2.0 > s8) ?: false) }, false))} \n' +
          '[1] scan s5 s6 none none none none lowPriority [s4 = date] @"f608ea24-04f3-4d10-ab13-ea4936e82c7e" true false '
      }
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 16864,
    executionTimeMillis: 202,
    totalKeysExamined: 0,
    totalDocsExamined: 384710,
    executionStages: {
      stage: 'filter',
      planNodeId: 1,
      nReturned: 16864,
      executionTimeMillisEstimate: 202,
      opens: 1,
      closes: 1,
      saveState: 384,
      restoreState: 384,
      isEOF: 1,
      numTested: 384710,
      filter: '(traverseF(s4, lambda(l1.0) { ((l1.0 < s7) ?: false) }, false) && traverseF(s4, lambda(l2.0) { ((l2.0 > s8) ?: false) }, false)) ',
      inputStage: {
        stage: 'scan',
        planNodeId: 1,
        nReturned: 384710,
        executionTimeMillisEstimate: 187,
        opens: 1,
        closes: 1,
        saveState: 384,
        restoreState: 384,
        isEOF: 1,
        numReads: 384710,
        recordSlot: 5,
        recordIdSlot: 6,
        fields: [ 'date' ],
        outputSlots: [ Long("4") ]
      }
    }
  },
  command: {
    find: 'calendar',
    filter: {
      '$and': [
        { date: { '$gt': '2023-10-15' } },
        { date: { '$lt': '2023-11-01' } }
      ]
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
mydb> db.calendar.createIndex({  date: 1})
date_1
mydb> db.calendar.find({ $and:  [{date:{ $gt: '2023-10-15'}}, {date:{ $lt: '2023-11-01'}}]}).explain("executionStats")
{
  explainVersion: '2',
  queryPlanner: {
    namespace: 'mydb.calendar',
    indexFilterSet: false,
    parsedQuery: {
      '$and': [
        { date: { '$lt': '2023-11-01' } },
        { date: { '$gt': '2023-10-15' } }
      ]
    },
    queryHash: '3D5C42CD',
    planCacheKey: '27D2418E',
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    winningPlan: {
      queryPlan: {
        stage: 'FETCH',
        planNodeId: 2,
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
          indexBounds: { date: [ '("2023-10-15", "2023-11-01")' ] }
        }
      },
      slotBasedPlan: {
        slots: '$$RESULT=s11 env: { s3 = 1697219245339 (NOW), s6 = KS(3C323032332D31312D3031000104), s5 = KS(3C323032332D31302D313500FE04), s2 = Nothing (SEARCH_META), s10 = {"date" : 1}, s1 = TimeZoneDatabase(America/Noronha...America/Danmarkshavn) (timeZoneDB) }',
        stages: '[2] nlj inner [] [s4, s7, s8, s9, s10] \n' +
          '    left \n' +
          '        [1] cfilter {(exists(s5) && exists(s6))} \n' +
          '        [1] ixseek s5 s6 s9 s4 s7 s8 [] @"f608ea24-04f3-4d10-ab13-ea4936e82c7e" @"date_1" true \n' +
          '    right \n' +
          '        [2] limit 1 \n' +
          '        [2] seek s4 s11 s12 s7 s8 s9 s10 [] @"f608ea24-04f3-4d10-ab13-ea4936e82c7e" true false \n'
      }
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 16864,
    executionTimeMillis: 58,
    totalKeysExamined: 16864,
    totalDocsExamined: 16864,
    executionStages: {
      stage: 'nlj',
      planNodeId: 2,
      nReturned: 16864,
      executionTimeMillisEstimate: 57,
      opens: 1,
      closes: 1,
      saveState: 16,
      restoreState: 16,
      isEOF: 1,
      totalDocsExamined: 16864,
      totalKeysExamined: 16864,
      collectionScans: 0,
      collectionSeeks: 16864,
      indexScans: 0,
      indexSeeks: 1,
      indexesUsed: [ 'date_1' ],
      innerOpens: 16864,
      innerCloses: 1,
      outerProjects: [],
      outerCorrelated: [ Long("4"), Long("7"), Long("8"), Long("9"), Long("10") ],
      outerStage: {
        stage: 'cfilter',
        planNodeId: 1,
        nReturned: 16864,
        executionTimeMillisEstimate: 52,
        opens: 1,
        closes: 1,
        saveState: 16,
        restoreState: 16,
        isEOF: 1,
        numTested: 1,
        filter: '(exists(s5) && exists(s6)) ',
        inputStage: {
          stage: 'ixseek',
          planNodeId: 1,
          nReturned: 16864,
          executionTimeMillisEstimate: 52,
          opens: 1,
          closes: 1,
          saveState: 16,
          restoreState: 16,
          isEOF: 1,
          indexName: 'date_1',
          keysExamined: 16864,
          seeks: 1,
          numReads: 16865,
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
        nReturned: 16864,
        executionTimeMillisEstimate: 5,
        opens: 16864,
        closes: 1,
        saveState: 16,
        restoreState: 16,
        isEOF: 1,
        limit: 1,
        inputStage: {
          stage: 'seek',
          planNodeId: 2,
          nReturned: 16864,
          executionTimeMillisEstimate: 5,
          opens: 16864,
          closes: 1,
          saveState: 16,
          restoreState: 16,
          isEOF: 0,
          numReads: 16864,
          recordSlot: 11,
          recordIdSlot: 12,
          seekKeySlot: 4,
          snapshotIdSlot: 7,
          indexIdentSlot: 8,
          indexKeySlot: 9,
          indexKeyPatternSlot: 10,
          fields: [],
          outputSlots: []
        }
      }
    }
  },
  command: {
    find: 'calendar',
    filter: {
      '$and': [
        { date: { '$gt': '2023-10-15' } },
        { date: { '$lt': '2023-11-01' } }
      ]
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
mydb>
