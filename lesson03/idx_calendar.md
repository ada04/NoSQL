Запрос

```nosql
db.calendar.find({ available: 'f'}).explain("executionStats")
```

Создание индекса

```
db.calendar.createIndex({  available: 1})
```

Без индекса

```sql
mydb> db.calendar.find({ available: 'f'}).explain("executionStats")
{
  explainVersion: '2',
  queryPlanner: {
    namespace: 'mydb.calendar',
    indexFilterSet: false,
    parsedQuery: { available: { '$eq': 'f' } },
    queryHash: 'EC5AA084',
    planCacheKey: 'DD554041',
    maxIndexedOrSolutionsReached: false,
    maxIndexedAndSolutionsReached: false,
    maxScansToExplodeReached: false,
    winningPlan: {
      queryPlan: {
        stage: 'COLLSCAN',
        planNodeId: 1,
        filter: { available: { '$eq': 'f' } },
        direction: 'forward'
      },
      slotBasedPlan: {
        slots: '$$RESULT=s5 env: { s2 = Nothing (SEARCH_META), s3 = 1697217981642 (NOW), s1 = TimeZoneDatabase(America/Noronha...America/Danmarkshavn) (timeZoneDB), s7 = "f" }',
        stages: '[1] filter {traverseF(s4, lambda(l1.0) { ((l1.0 == s7) ?: false) }, false)} \n' +
          '[1] scan s5 s6 none none none none lowPriority [s4 = available] @"f608ea24-04f3-4d10-ab13-ea4936e82c7e" true false '
      }
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 189856,
    executionTimeMillis: 174,
    totalKeysExamined: 0,
    totalDocsExamined: 384710,
    executionStages: {
      stage: 'filter',
      planNodeId: 1,
      nReturned: 189856,
      executionTimeMillisEstimate: 174,
      opens: 1,
      closes: 1,
      saveState: 384,
      restoreState: 384,
      isEOF: 1,
      numTested: 384710,
      filter: 'traverseF(s4, lambda(l1.0) { ((l1.0 == s7) ?: false) }, false) ',
      inputStage: {
        stage: 'scan',
        planNodeId: 1,
        nReturned: 384710,
        executionTimeMillisEstimate: 168,
        opens: 1,
        closes: 1,
        saveState: 384,
        restoreState: 384,
        isEOF: 1,
        numReads: 384710,
        recordSlot: 5,
        recordIdSlot: 6,
        fields: [ 'available' ],
        outputSlots: [ Long("4") ]
      }
    }
  },
  command: { find: 'calendar', filter: { available: 'f' }, '$db': 'mydb' },
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

Создаем индекс и повторяем запрос

```
mydb> db.calendar.createIndex({  available: 1})
available_1
mydb> db.calendar.find({ available: 'f'}).explain("executionStats")
{
  explainVersion: '2',
  queryPlanner: {
    namespace: 'mydb.calendar',
    indexFilterSet: false,
    parsedQuery: { available: { '$eq': 'f' } },
    queryHash: 'EC5AA084',
    planCacheKey: '07111A2C',
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
          keyPattern: { available: 1 },
          indexName: 'available_1',
          isMultiKey: false,
          multiKeyPaths: { available: [] },
          isUnique: false,
          isSparse: false,
          isPartial: false,
          indexVersion: 2,
          direction: 'forward',
          indexBounds: { available: [ '["f", "f"]' ] }
        }
      },
      slotBasedPlan: {
        slots: '$$RESULT=s11 env: { s2 = Nothing (SEARCH_META), s5 = KS(3C66000104), s10 = {"available" : 1}, s3 = 1697218102522 (NOW), s6 = KS(3C6600FE04), s1 = TimeZoneDatabase(America/Noronha...America/Danmarkshavn) (timeZoneDB) }',
        stages: '[2] nlj inner [] [s4, s7, s8, s9, s10] \n' +
          '    left \n' +
          '        [1] cfilter {(exists(s5) && exists(s6))} \n' +
          '        [1] ixseek s5 s6 s9 s4 s7 s8 [] @"f608ea24-04f3-4d10-ab13-ea4936e82c7e" @"available_1" true \n' +
          '    right \n' +
          '        [2] limit 1 \n' +
          '        [2] seek s4 s11 s12 s7 s8 s9 s10 [] @"f608ea24-04f3-4d10-ab13-ea4936e82c7e" true false \n'
      }
    },
    rejectedPlans: []
  },
  executionStats: {
    executionSuccess: true,
    nReturned: 189856,
    executionTimeMillis: 267,
    totalKeysExamined: 189856,
    totalDocsExamined: 189856,
    executionStages: {
      stage: 'nlj',
      planNodeId: 2,
      nReturned: 189856,
      executionTimeMillisEstimate: 260,
      opens: 1,
      closes: 1,
      saveState: 189,
      restoreState: 189,
      isEOF: 1,
      totalDocsExamined: 189856,
      totalKeysExamined: 189856,
      collectionScans: 0,
      collectionSeeks: 189856,
      indexScans: 0,
      indexSeeks: 1,
      indexesUsed: [ 'available_1' ],
      innerOpens: 189856,
      innerCloses: 1,
      outerProjects: [],
      outerCorrelated: [ Long("4"), Long("7"), Long("8"), Long("9"), Long("10") ],
      outerStage: {
        stage: 'cfilter',
        planNodeId: 1,
        nReturned: 189856,
        executionTimeMillisEstimate: 247,
        opens: 1,
        closes: 1,
        saveState: 189,
        restoreState: 189,
        isEOF: 1,
        numTested: 1,
        filter: '(exists(s5) && exists(s6)) ',
        inputStage: {
          stage: 'ixseek',
          planNodeId: 1,
          nReturned: 189856,
          executionTimeMillisEstimate: 247,
          opens: 1,
          closes: 1,
          saveState: 189,
          restoreState: 189,
          isEOF: 1,
          indexName: 'available_1',
          keysExamined: 189856,
          seeks: 1,
          numReads: 189857,
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
        nReturned: 189856,
        executionTimeMillisEstimate: 12,
        opens: 189856,
        closes: 1,
        saveState: 189,
        restoreState: 189,
        isEOF: 1,
        limit: 1,
        inputStage: {
          stage: 'seek',
          planNodeId: 2,
          nReturned: 189856,
          executionTimeMillisEstimate: 12,
          opens: 189856,
          closes: 1,
          saveState: 189,
          restoreState: 189,
          isEOF: 0,
          numReads: 189856,
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
  command: { find: 'calendar', filter: { available: 'f' }, '$db': 'mydb' },
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
