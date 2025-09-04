# Solr Admin Console Guide
Comprehensive overview of managing Solr 8.11 through the Administrative Console interface

## Prerequisites
- Solr 8.11 installed as outlined in the **[Apache Solr Reference Guide](https://solr.apache.org/guide/8_11/)**

## Assumptions
- Your project uses a custom configuration, similar to the setup defined in the [Basic Setup Guide](BASIC%20SETUP.md) in favor of Solr's default configuration, implemented through use of the ```managed-schema``` file.
---

## Table of Contents

### **[Access the Admin Console](#access-the-admin-console)**
 - [Verify Access](#verify-access)

### **[Console Navigation](#console-navigation)**
- [Main Navigation Areas](#main-navigation-areas)
- [Core-Specific Navigation](#core-specific-navigation)

### **[Understanding the Dashboard](#understanding-the-dashboard)**
- [System Information](#system-information)
- [Core Overview](#core-overview)

### **[Core Management](#core-management)**
- [Core Administration](#core-administration)
- [Creating New Cores](#creating-new-cores)

### **[Document Management](#document-management)**
- [Viewing Documents](#viewing-documents)
- [Adding Documents](#adding-documents)
- [Updating Documents](#updating-documents)
- [Deleting Documents](#deleting-documents)

### **[Schema Management](#schema-management)**
- [Viewing Schema Information](#viewing-schema-information)
- [Managed Schema Modifications](#managed-schema-modifications)
- [Traditional Schema Modifications](#traditional-schema-modifications)
- [Schema Browser Features](#schema-browser-features)

### **[Query Interface and Testing](#query-interface-and-testing)**
- [Query Builder Interface](#query-builder-interface)
- [Common Query Examples](#common-query-examples)
- [Query Testing and Debugging](#query-testing-and-debugging)
- [Faceting and Aggregation](#faceting-and-aggregation)

### **[Performance Monitoring](#performance-monitoring)**
- [Performance Metrics Dashboard](#performance-metrics-dashboard)
- [Query Performance Analysis](#query-performance-analysis)
- [Cache Statistics](#cache-statistics)
- [Index Statistics](#index-statistics)

### **[Log Management](#log-management)**
- [Accessing Logs via Admin Console](#accessing-logs-via-admin-console)
- [Log Level Management](#log-level-management)
- [Common Log Patterns](#common-log-patterns)
- [Log File Locations](#log-file-locations)

### **[Common Administrative Tasks](#common-administrative-tasks)**
- [Daily Maintenance Tasks](#daily-maintenance-tasks)
- [Index Optimization](#index-optimization)
- [Core Backup and Restore](#core-backup-and-restore)
- [Configuration Management](#configuration-management)

---

## Access the Admin Console:
- Open your browser to: **http://localhost:8983/solr/**
- Default port is 8983 (unless you've configured differently)
- No authentication required for local development

**Verify Access:**
- You should see the Solr Admin Console dashboard
- Left sidebar shows available cores
- Main area shows dashboard information

## Console Navigation
### **Main Navigation Areas:**
- **Dashboard**: Overview and system information
- **Logging**: Real-time log viewing and level management
- **Core Admin**: Core-specific management
- **Java Properties**: System properties and configuration
- **Thread Dump**: System diagnostics

### **Core-Specific Navigation:**
- **Overview**: Core statistics and information
- **Analysis**: Text analysis and field processing
- **Documents**: Document management interface
- **Query**: Search interface and query testing
- **Schema**: Schema viewing and modification
- **Config**: Configuration file viewing
- **Replication**: Replication status and management
- **Distribution**: SolrCloud distribution information

## Understanding the Dashboard

### System Information

**Dashboard Overview:**
```
Solr Admin Console Dashboard Elements:

┌─ System Information ─────────────────────────────┐
│ • Solr Version: 8.11.2                          │
│ • Java Version: 11.0.x                          │
│ • Uptime: 2 days, 4 hours                       │
│ • Memory Usage: 512MB / 2GB                     │
└──────────────────────────────────────────────────┘

┌─ Instance Information ───────────────────────────┐
│ • Instance Directory: /path/to/solr/server      │
│ • Data Directory: /path/to/solr/server/solr     │
│ • Config Directory: solr/configsets             │
└──────────────────────────────────────────────────┘
```

**Key Metrics to Monitor:**
- **Memory Usage**: Should stay below 80% of allocated heap
- **Document Count**: Total documents across all cores
- **Query Rate**: Queries per second
- **Index Size**: Total disk space used

### Core Overview

**Accessing Core Dashboard:**
1. Click on your core name (e.g., `myapp_development`) in the left sidebar
2. Click **"Overview"** to see core-specific statistics

**Core Statistics Display:**
```
Core: myapp_development
├─ Index
│  ├─ Num Docs: 20,450
│  ├─ Max Doc: 20,450  
│  ├─ Deleted Docs: 0
│  ├─ Index Size: 45.2 MB
│  └─ Last Modified: 2024-01-15T10:30:00Z
├─ Cache Statistics
│  ├─ Document Cache: 85% hit ratio
│  ├─ Query Result Cache: 92% hit ratio
│  └─ Filter Cache: 78% hit ratio
└─ Request Statistics
   ├─ Total Requests: 1,247
   ├─ Average Response Time: 23ms
   └─ Errors: 2
```

---

## Core Management

### Core Administration

**Accessing Core Admin:**
1. Go to **Core Admin** in the left sidebar
2. Select your core from the dropdown

**Core Operations:**
```
Available Core Actions:

┌─ Core Management ────────────────────────────────┐
│ • Reload: Refresh configuration without restart  │
│ • Rename: Change core name                       │
│ • Swap: Exchange two cores                       │
│ • Unload: Remove core from memory               │
│ • Create: Add new core                          │
│ • Split: Divide core into multiple cores        │
│ • Merge: Combine multiple cores                 │
└──────────────────────────────────────────────────┘
```

**Reloading a Core (Most Common):**
1. Select your core: `myapp_development`
2. Click **"Reload"** button
3. Check for success message or errors

**When to Reload:**
- After schema.xml changes (traditional schema)
- After solrconfig.xml modifications
- When configuration changes aren't taking effect
- After adding new field types or request handlers

### Creating New Cores

**Creating a Core via Admin Console:**
1. Go to **Core Admin**
2. Click **"Add Core"**
3. Fill in the form:

```
Add Core Form:
┌────────────────────────────────────────────────┐
│ name: myapp_test                               │
│ instanceDir: myapp_test                        │
│ dataDir: data                                  │
│ config: solrconfig.xml                         │
│ schema: schema.xml                             │
│ configSet: _default (or custom configset)     │
└────────────────────────────────────────────────┘
```

4. Click **"Add Core"**

**Core Creation Best Practices:**
- Use descriptive names (`myapp_development`, `myapp_production`)
- Keep consistent directory structure
- Copy configuration from working cores
- Test with small datasets first

---

## Document Management

### Viewing Documents

**Accessing Document Interface:**
1. Select your core: `myapp_development`
2. Click **"Documents"** in the core menu

**Document Interface Overview:**
```
Documents Interface:
┌─ Document Operations ────────────────────────────┐
│ • Document Type: JSON/XML/CSV                    │
│ • Request Handler: /update                       │
│ • Document(s): [Text area for document content]  │
│ • Commit Within: 1000ms                         │
│ • Overwrite: true                               │
│ • Boost: 1.0                                    │
└──────────────────────────────────────────────────┘
```

**Viewing Existing Documents:**
1. Go to **Query** interface
2. Use query: `*:*` to see all documents
3. Set **rows** parameter to limit results
4. Click **"Execute Query"**

**Sample Query Results:**
```json
{
  "response": {
    "numFound": 20450,
    "start": 0,
    "docs": [
      {
        "id": "person_12345",
        "given_name_tesim": ["John"],
        "family_name_tesim": ["Doe"],
        "full_name_tesim": ["John Doe"],
        "type_sim": ["Person"],
        "_version_": 1234567890
      }
    ]
  }
}
```

### Adding Documents

**Adding Single Document:**
1. Go to **Documents** interface
2. Select **Document Type**: `JSON`
3. Enter document in text area:

```json
{
  "id": "test_document_001",
  "title_tesim": ["Test Document"],
  "description_tesim": ["This is a test document"],
  "type_sim": ["Document"],
  "creator_sim": ["Admin"],
  "year_isim": [2024]
}
```

4. Click **"Submit Document"**

**Adding Multiple Documents:**
```json
[
  {
    "id": "doc1",
    "title_tesim": ["First Document"],
    "type_sim": ["Document"]
  },
  {
    "id": "doc2", 
    "title_tesim": ["Second Document"],
    "type_sim": ["Document"]
  }
]
```

**Important Document Fields:**
- **id**: Must be unique across the core
- **Field naming**: Must match schema field definitions
- **Multi-valued fields**: Use arrays for `multiValued="true"` fields
- **Field types**: Ensure data types match schema definitions

### Updating Documents

**Updating Existing Document:**
1. Add document with same **id** as existing document
2. Set **Overwrite**: `true` (default)
3. Include all fields you want to keep (partial updates require atomic updates)

**Atomic Updates:**
```json
{
  "id": "person_12345",
  "given_name_tesim": {"set": "Jane"},
  "year_isim": {"add": 1985}
}
```

**Atomic Update Operations:**
- `"set"`: Replace field value
- `"add"`: Add to numeric field or multi-valued field
- `"remove"`: Remove specific value from multi-valued field
- `"removeregex"`: Remove values matching regex
- `"inc"`: Increment numeric field

### Deleting Documents

**Delete by ID:**
1. Go to **Documents** interface
2. Select Document Type: `XML`
3. Enter delete command:

```xml
<delete>
  <id>test_document_001</id>
</delete>
```

4. Click **"Submit Document"**

**Delete by Query:**
```xml
<delete>
  <query>type_sim:Document AND year_isim:[2020 TO 2022]</query>
</delete>
```

**Delete All Documents:**
```xml
<delete>
  <query>*:*</query>
</delete>
```

**Committing Changes:**
After adding/updating/deleting documents:
```xml
<commit/>
```

Or enable auto-commit by setting **Commit Within** to a value like `1000` (1 second).

---

## Schema Management

### Viewing Schema Information

**Accessing Schema Interface:**
1. Select your core: `myapp_development`
2. Click **"Schema"** in the core menu

**Schema Browser Navigation:**
```
Schema Browser Sections:
┌─ Schema Information ─────────────────────────────┐
│ • Overview: Schema summary and statistics        │
│ • Fields: All defined fields                     │ 
│ • Dynamic Fields: Pattern-based field definitions│
│ • Field Types: Data type definitions            │
│ • Copy Fields: Field copying rules              │
│ • Analyzers: Text analysis chains               │
└──────────────────────────────────────────────────┘
```

**Field Information Display:**
```
Field: given_name_tesim
├─ Type: text_general
├─ Properties:
│  ├─ Indexed: true
│  ├─ Stored: true  
│  ├─ Multi-Valued: true
│  ├─ Required: false
│  └─ Doc Values: false
├─ Analysis Chain:
│  ├─ Tokenizer: StandardTokenizerFactory
│  ├─ Filters: 
│  │  ├─ StopFilterFactory
│  │  └─ LowerCaseFilterFactory
└─ Statistics:
   ├─ Documents with field: 5,247
   └─ Unique values: 3,891
```

### Managed Schema Modifications

**For Managed Schema Cores:**

**Adding Field via API:**
1. Go to **Schema** interface
2. Click **"Add Field"** button
3. Fill in the form:

```
Add Field Form:
┌────────────────────────────────────────────────┐
│ Name: new_field_tesim                          │
│ Type: text_general                             │
│ □ Stored      ☑ Indexed                       │
│ ☑ Multi-Valued  □ Required                    │
│ □ Doc Values   □ Use Doc Values as Stored     │
│ Default Value: [empty]                         │
└────────────────────────────────────────────────┘
```

4. Click **"Add Field"**

**Adding Field Type:**
1. Click **"Add Field Type"**
2. Define field type:

```json
{
  "name": "custom_text",
  "class": "solr.TextField",
  "analyzer": {
    "tokenizer": {
      "class": "solr.StandardTokenizerFactory"
    },
    "filters": [
      {
        "class": "solr.LowerCaseFilterFactory"
      }
    ]
  }
}
```

**Adding Copy Field:**
1. Click **"Add Copy Field"**
2. Specify source and destination:

```
Copy Field Form:
┌────────────────────────────────────────────────┐
│ Source: title_tesim                            │
│ Destination: searchable_text_tesim             │
└────────────────────────────────────────────────┘
```

### Traditional Schema Modifications

**For Traditional Schema (schema.xml):**

**Viewing Current Schema:**
1. Go to **Files** in core menu
2. Select `schema.xml`
3. View current configuration

**Making Changes:**
1. Stop Solr: `./solr stop`
2. Edit `solr/server/solr/myapp_development/conf/schema.xml`
3. Add/modify field definitions:

```xml
<!-- Add new field -->
<field name="new_field_tesim" type="text_general" 
       indexed="true" stored="true" multiValued="true"/>

<!-- Add new field type -->
<fieldType name="custom_text" class="solr.TextField">
  <analyzer>
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
</fieldType>

<!-- Add copy field -->
<copyField source="title_tesim" dest="searchable_text_tesim"/>
```

4. Start Solr: `./solr start`
5. Reload core in admin console

**Schema Validation:**
1. After changes, check **Schema** interface for errors
2. Look for red error messages or warnings
3. Check Solr logs for validation issues

### Schema Browser Features

**Field Analysis Tool:**
1. Go to **Analysis** in core menu
2. Test field processing:

```
Analysis Interface:
┌─ Field Analysis ─────────────────────────────────┐
│ Field Value (Index): John Doe                    │
│ Field Type: text_general                         │
│                                                  │
│ Analysis Results:                                │
│ StandardTokenizer: [John] [Doe]                  │
│ StopFilterFactory: [John] [Doe]                  │
│ LowerCaseFilterFactory: [john] [doe]             │
└──────────────────────────────────────────────────┘
```

**Schema Browser Filters:**
- **Filter by Name**: Search for specific fields
- **Filter by Type**: Show fields of specific type
- **Show Only**: Dynamic fields, stored fields, indexed fields
- **Sort Options**: Alphabetical, by type, by usage

---

## Query Interface and Testing

### Query Builder Interface

**Accessing Query Interface:**
1. Select your core: `myapp_development`
2. Click **"Query"** in the core menu

**Query Form Parameters:**
```
Query Interface:
┌─ Request Handler ────────────────────────────────┐
│ Request Handler: /select                         │
└──────────────────────────────────────────────────┘

┌─ Common Parameters ──────────────────────────────┐
│ q: *:*                                          │
│ fq: [Filter Query]                              │
│ sort: score desc                                │
│ start,rows: 0,10                               │
│ fl: [Field List]                               │
│ df: [Default Field]                            │
│ Raw Query Parameters: [Additional params]       │
│ wt: json                                       │
└──────────────────────────────────────────────────┘

┌─ Highlighting ───────────────────────────────────┐
│ hl: false                                       │
│ hl.fl: [Highlight Fields]                      │
└──────────────────────────────────────────────────┘

┌─ Faceting ───────────────────────────────────────┐
│ facet: false                                    │
│ facet.field: [Facet Fields]                    │
│ facet.query: [Facet Queries]                   │
└──────────────────────────────────────────────────┘

┌─ Spatial ────────────────────────────────────────┐
│ pt: [Point]                                     │
│ d: [Distance]                                   │
│ spatial: [Spatial Options]                     │
└──────────────────────────────────────────────────┘
```

### Common Query Examples

**Basic Queries:**

```bash
# Find all documents
q: *:*

# Search in specific field
q: given_name_tesim:John

# Multiple field search
q: given_name_tesim:John OR family_name_tesim:Doe

# Phrase search
q: full_name_tesim:"John Doe"

# Wildcard search
q: given_name_tesim:Joh*

# Range search
q: year_isim:[1990 TO 2000]
```

**Advanced Queries:**

```bash
# Boolean operators
q: given_name_tesim:John AND family_name_tesim:Doe

# Field existence
q: _exists_:email_tesim

# Missing field
q: -_exists_:email_tesim

# Boosting
q: given_name_tesim:John^2 OR family_name_tesim:Doe

# Proximity search
q: description_tesim:"emergency response"~5
```

**Filter Queries (fq):**

```bash
# Single filter
fq: type_sim:Person

# Multiple filters
fq: type_sim:Person
fq: year_isim:[1980 TO 2000]

# Complex filter
fq: (type_sim:Person OR type_sim:Document) AND year_isim:[2020 TO *]
```

### Query Testing and Debugging

**Testing Search Performance:**
1. Use **debugQuery**: `true` in Raw Query Parameters
2. Check timing information in response
3. Monitor **Query Time** in results

**Debug Query Output:**
```json
{
  "debug": {
    "querystring": "given_name_tesim:John",
    "parsedquery": "given_name_tesim:john",
    "explain": {
      "person_12345": {
        "match": true,
        "value": 2.3,
        "description": "weight(given_name_tesim:john)"
      }
    },
    "timing": {
      "time": 12.5,
      "prepare": {
        "time": 2.1,
        "query": {"time": 1.8},
        "facet": {"time": 0.3}
      },
      "process": {
        "time": 10.4,
        "query": {"time": 8.2},
        "facet": {"time": 2.2}
      }
    }
  }
}
```

**Query Analysis Tools:**
- **explain**: Shows scoring details
- **debugQuery**: Detailed query analysis
- **debug.explain.structured**: Structured scoring explanation
- **timeAllowed**: Set query timeout

### Faceting and Aggregation

**Basic Faceting:**
```bash
# Enable faceting
facet: true

# Field facets
facet.field: type_sim
facet.field: year_isim

# Range facets
facet.range: year_isim
facet.range.start: 1900
facet.range.end: 2025
facet.range.gap: 10
```

**Query Facets:**
```bash
facet.query: year_isim:[1990 TO 1999]
facet.query: year_isim:[2000 TO 2009]
facet.query: year_isim:[2010 TO 2019]
```

**Pivot Facets:**
```bash
facet.pivot: type_sim,year_isim
facet.pivot: creator_sim,type_sim
```

## Performance Monitoring

### Performance Metrics Dashboard

**Key Performance Indicators:**

```
Performance Dashboard:
┌─ Query Performance ──────────────────────────────┐
│ • Average Query Time: 23ms                      │
│ • 95th Percentile: 45ms                         │
│ • 99th Percentile: 120ms                        │
│ • Slow Queries (>1s): 3                         │
│ • Total Queries: 12,487                         │
│ • Queries/Second: 14.2                          │
└──────────────────────────────────────────────────┘

┌─ Cache Performance ──────────────────────────────┐
│ • Document Cache Hit Ratio: 85%                 │
│ • Query Result Cache Hit Ratio: 92%             │
│ • Filter Cache Hit Ratio: 78%                   │
│ • Field Value Cache Hit Ratio: 96%              │
└──────────────────────────────────────────────────┘

┌─ Memory Usage ───────────────────────────────────┐
│ • Heap Memory: 512MB / 2GB (25%)                │
│ • Non-Heap Memory: 128MB / 256MB (50%)          │
│ • Direct Memory: 64MB / 128MB (50%)             │
│ • GC Activity: 12 collections, 2.3s total      │
└──────────────────────────────────────────────────┘
```

### Query Performance Analysis

**Using the Query Interface for Performance Testing:**

1. **Basic Performance Test:**
```bash
# Test query performance
q: *:*
rows: 100
debugQuery: true

# Check timing in debug output
```

2. **Complex Query Performance:**
```bash
# Test faceted search performance
q: searchable_text_tesim:search_term
facet: true
facet.field: type_sim
facet.field: creator_sim
facet.field: year_isim
debugQuery: true
```

3. **Large Result Set Performance:**
```bash
# Test pagination performance
q: type_sim:Person
start: 5000
rows: 50
```

**Performance Timing Breakdown:**
```json
{
  "debug": {
    "timing": {
      "time": 45.2,
      "prepare": {
        "time": 5.1,
        "query": {"time": 3.2},
        "facet": {"time": 1.9}
      },
      "process": {
        "time": 40.1,
        "query": {"time": 25.3},
        "facet": {"time": 14.8}
      }
    }
  }
}
```

### Cache Statistics

**Viewing Cache Performance:**
1. Go to **Plugins/Stats** in core menu
2. Click on **CACHE** category
3. Review cache statistics:

```
Cache Statistics:
┌─ filterCache ────────────────────────────────────┐
│ • lookups: 12,487                               │
│ • hits: 9,740 (78%)                            │
│ • hitratio: 0.78                               │
│ • inserts: 2,747                               │
│ • evictions: 12                                │
│ • size: 485                                    │
│ • warmupTime: 0                                │
└──────────────────────────────────────────────────┘

┌─ queryResultCache ───────────────────────────────┐
│ • lookups: 8,234                                │
│ • hits: 7,575 (92%)                            │
│ • hitratio: 0.92                               │
│ • inserts: 659                                 │
│ • evictions: 3                                 │
│ • size: 187                                    │
│ • warmupTime: 45                               │
└──────────────────────────────────────────────────┘
```

**Cache Optimization:**
- **Document Cache**: Should have >80% hit ratio
- **Query Result Cache**: Should have >85% hit ratio
- **Filter Cache**: Should have >75% hit ratio
- **High evictions**: Indicate cache size too small

### Index Statistics

**Index Health Monitoring:**
1. Go to **Overview** for your core
2. Check index statistics:

```
Index Statistics:
┌─ Index Size and Documents ───────────────────────┐
│ • Number of Documents: 20,450                    │
│ • Maximum Document: 20,450                       │
│ • Deleted Documents: 0                           │
│ • Index Size on Disk: 45.2 MB                   │
│ • Segments: 12                                   │
│ • Index Version: 1234567890                     │
│ • Last Modified: 2024-01-15T10:30:00Z           │
└──────────────────────────────────────────────────┘

┌─ Segment Information ────────────────────────────┐
│ • Segment Count: 12                              │
│ • Total Size: 45.2 MB                           │
│ • Largest Segment: 8.3 MB                       │
│ • Optimization Needed: Yes (>10 segments)       │
└──────────────────────────────────────────────────┘
```

**Index Optimization Indicators:**
- **Many segments (>20)**: Consider running optimize
- **High deleted documents**: Run expungeDeletes
- **Large index size**: Review data and compression
- **Frequent updates**: Tune merge policies

---

## Log Management

### Accessing Logs via Admin Console

**Log Viewing Interface:**
1. Click **"Logging"** in the main navigation
2. View real-time log entries
3. Filter by log level and logger

**Log Interface Features:**
```
Logging Interface:
┌─ Log Level Controls ─────────────────────────────┐
│ • Level: [ALL|TRACE|DEBUG|INFO|WARN|ERROR|FATAL] │
│ • Since: [Time filter]                           │
│ • Logger: [Specific logger filter]               │
└──────────────────────────────────────────────────┘

┌─ Log Entries ────────────────────────────────────┐
│ 2024-01-15 10:30:15 INFO  [Request Handler]     │
│ Query executed: q=*:* in 23ms                    │
│                                                  │
│ 2024-01-15 10:30:10 WARN  [Schema]              │
│ Unknown field 'bad_field_name' in document      │
│                                                  │
│ 2024-01-15 10:30:05 ERROR [Update Processor]    │
│ Failed to parse document: invalid JSON          │
└──────────────────────────────────────────────────┘
```

### Log Level Management

**Changing Log Levels:**
1. Go to **Logging** interface
2. Find logger in the **Logger Settings** section
3. Change level for specific loggers:

```
Logger Level Settings:
┌─ Common Loggers ─────────────────────────────────┐
│ • org.apache.solr.core: INFO                    │
│ • org.apache.solr.update: WARN                  │
│ • org.apache.solr.search: INFO                  │
│ • org.apache.solr.request: DEBUG                │
│ • org.apache.solr.handler: INFO                 │
│ • org.apache.lucene: WARN                       │
└──────────────────────────────────────────────────┘
```

**Useful Log Levels for Debugging:**
- **DEBUG**: Detailed execution information
- **INFO**: General operation information
- **WARN**: Potential issues and warnings
- **ERROR**: Error conditions
- **TRACE**: Very detailed debugging (use sparingly)

### Common Log Patterns

**Successful Operations:**
```
INFO  [UpdateHandler] Added document: person_12345
INFO  [QueryHandler] Query: q=name:john rows=10 hits=5 time=23ms
INFO  [CommitHandler] Committed in 145ms
```

**Warning Conditions:**
```
WARN  [SchemaHandler] Unknown field 'bad_field' ignored
WARN  [CacheHandler] High eviction rate in filterCache: 15%
WARN  [UpdateHandler] Document overwrite: person_12345
```

**Error Conditions:**
```
ERROR [UpdateHandler] Failed to parse document: invalid JSON at line 5
ERROR [CoreHandler] Schema reload failed: invalid field type definition
ERROR [SearchHandler] Query timeout after 30000ms: q=complex_query
```

### Log File Locations

**Physical Log Files:**
```bash
# Main Solr log
tail -f solr/server/logs/solr.log

# Garbage collection logs  
tail -f solr/server/logs/solr_gc.log

# Console output
tail -f solr/server/logs/solr-8983-console.log
```

**Log Rotation:**
- Logs rotate daily by default
- Keep last 30 days of logs
- Configure in `solr/server/resources/log4j2.xml`

---

## Common Administrative Tasks

### Daily Maintenance Tasks

**Daily Health Check Workflow:**

```bash
Daily Solr Maintenance Checklist:

┌─ System Health ──────────────────────────────────┐
│ ☐ Check Solr is running (Admin Console access)  │
│ ☐ Verify all cores are loaded                   │
│ ☐ Check memory usage (<80% of heap)             │
│ ☐ Review error logs for issues                  │
│ ☐ Monitor query response times                  │
│ ☐ Check cache hit ratios                        │
└──────────────────────────────────────────────────┘

┌─ Data Integrity ─────────────────────────────────┐
│ ☐ Verify document counts match expectations     │
│ ☐ Check for deleted documents                   │
│ ☐ Monitor index size growth                     │
│ ☐ Review recent update operations               │
└──────────────────────────────────────────────────┘
```

**Using Admin Console for Daily Checks:**

1. **System Overview:**
    - Check **Dashboard** for system status
    - Monitor memory usage and uptime
    - Review instance information

2. **Core Health:**
    - Visit each core's **Overview**
    - Check document counts and index size
    - Review cache statistics

3. **Query Performance:**
    - Use **Query** interface to test common searches
    - Monitor response times
    - Check for query errors

### Index Optimization

**When to Optimize:**
- More than 20 segments in index
- High percentage of deleted documents (>10%)
- Noticeable search performance degradation
- After large batch updates

**Running Optimization:**

**Via Admin Console:**
1. Go to **Documents** interface
2. Select Document Type: **XML**
3. Enter optimize command:
```xml
<optimize/>
```
4. Click **"Submit Document"**

**Via Direct URL:**
```bash
# Optimize specific core
curl "http://localhost:8983/solr/myapp_development/update?optimize=true"

# Optimize with specific parameters
curl "http://localhost:8983/solr/myapp_development/update?optimize=true&maxSegments=1&waitSearcher=true"
```

**Optimization Monitoring:**
- Watch **Overview** for segment count reduction
- Monitor index size changes
- Check logs for optimization completion
- Test query performance before/after

### Core Backup and Restore

**Creating Backups via Admin Console:**

1. **Backup Request:**
    - Go to **Replication** in core menu
    - Click **"Backup"** section
    - Specify backup parameters:

```
Backup Parameters:
┌────────────────────────────────────────────────┐
│ Command: backup                                │
│ Name: backup_2024_01_15                        │
│ Location: /path/to/backup/directory            │
│ Repository: [optional repository name]         │
│ Async ID: backup_job_001                       │
└────────────────────────────────────────────────┘
```

2. **Monitor Backup Progress:**
    - Check **Replication** status
    - Monitor async request status
    - Watch filesystem for backup creation

**Restore from Backup:**
1. Go to **Replication** interface
2. Use **"Restore"** section:

```
Restore Parameters:
┌────────────────────────────────────────────────┐
│ Command: restore                               │
│ Name: backup_2024_01_15                        │
│ Location: /path/to/backup/directory            │
│ Async ID: restore_job_001                      │
└────────────────────────────────────────────────┘
```

### Configuration Management

**Viewing Configuration Files:**
1. Go to **Files** in core menu
2. Browse configuration files:
    - `solrconfig.xml` - Core configuration
    - `schema.xml` or `managed-schema` - Schema definition
    - `stopwords.txt` - Stop words list
    - `synonyms.txt` - Synonym definitions

**Configuration File Editor:**
- Click on file name to view
- Some files are editable directly in interface
- Changes require core reload

**Configuration Validation:**
1. Make configuration changes
2. Reload core via **Core Admin**
3. Check for validation errors
4. Test functionality with sample queries

---

note: This guide was adapted from a Claude AI prompt. Most steps have been tested and confirmed as accurate for Solr 8.11, but please do submit a PR for any errors found.
