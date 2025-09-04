# Solr Troubleshooting Guide
Steps to diagnose and resolve common issues

## Prerequisites
- Solr 8.11 installed as outlined in the **[Apache Solr Reference Guide](https://solr.apache.org/guide/8_11/)**
---

## Quick Reference Guide
- **Issues with search**: start with [Document Indexing Issues](#document-indexing-issues)
- **Slow query response time and/or memory usage spikes**: Go to [Performance Issues](#performance-issues)
- **Schema field errors**: [Schema Problems](#schema-problems)
- **Solr Core**: [Connection and Core Issues](#connection-and-core-issues)

---

## Table of Contents
### [Document Indexing Issues](#document-indexing-issues)
#### *Problem: Documents not appearing in search results*
- [Verify Document Was Added](#verify-document-was-added)
- [Check Document Structure](#check-document-structure)
- [Verify Schema Match](#verify-schema-match)
- [Test Field Analysis](#test-field-analysis)
- [Commit Status](#commit-status)
#### *Problem: Search results don't match expectations*
- [Test Simple Query](#test-simple-query)
- [Use Debug Query](#use-debug-query)
- [Check Field Analysis](#check-field-analysis)
- [Review Scoring](#review-scoring)

### [Performance Issues](#performance-issues)
#### *Problem: Slow query performance*
- [Query Timing Analysis](#query-timing-analysis)
- [Check Cache Performance](#check-cache-performance)
- [Index Analysis](#index-analysis)
- [Query Complexity](#query-complexity)
#### *Problem: High memory usage*
- [Check Memory Statistics](#check-memory-statistics)
- [Cache Analysis](#cache-analysis)
- [Index Size](#index-size)

### [Schema Problems](#schema-problems)
#### *Problem: Unknown field errors*
- [Verify Field Definition](#verify-field-definition)
- [For Managed Schema](#for-managed-schema)
- [For Traditional Schema](#for-traditional-schema)
- [Test Field Addition](#test-field-addition)
#### *Problem: Schema reload failures*
- [Check Syntax](#check-syntax)
- [Review Error Messages](#review-error-messages)
- [Test Incremental Changes](#test-incremental-changes)

### [Connection and Core Issues](#connection-and-core-issues)
#### *Problem: Core not accessible*
- [Check Core Status](#check-core-status)
- [Core Reload](#core-reload)
- [Configuration Validation](#configuration-validation)
- [Core Recreation](#core-recreation)
#### *Problem: Solr not responding*
- [Check Solr Process](#check-solr-process)
- [Review System Resources](#review-system-resources)
- [Log Analysis](#log-analysis)
- [Restart Procedure](#restart-procedure)

---

## Troubleshooting Workflows

### Document Indexing Issues

**Problem: Documents not appearing in search results**

**Troubleshooting Steps:**

1. **Verify Document Was Added:**
```bash
# Query by specific ID
q: id:your_document_id
```

2. **Check Document Structure:**
    - Go to **Documents** interface
    - Review document JSON/XML format
    - Verify required fields are present
    - Check field name spelling

3. **Verify Schema Match:**
    - Go to **Schema** interface
    - Confirm fields exist in schema
    - Check field types match data types
    - Verify multi-valued settings

4. **Test Field Analysis:**
    - Go to **Analysis** interface
    - Test field processing with sample data
    - Check tokenization and filtering

5. **Commit Status:**
```xml
<!-- Ensure documents are committed -->
<commit/>
```

**Problem: Search results don't match expectations**

**Diagnosis Workflow:**

1. **Test Simple Query:**
```bash
# Start with wildcard search
q: *:*

# Test specific field
q: field_name:search_term
```

2. **Use Debug Query:**
```bash
q: your_search_query
debugQuery: true
```

3. **Check Field Analysis:**
    - Compare index analysis vs query analysis
    - Verify stemming and filtering effects
    - Test with exact field values

4. **Review Scoring:**
    - Use `explain=true` parameter
    - Check field boosting
    - Verify relevance scoring

### Performance Issues

**Problem: Slow query performance**

**Performance Analysis Steps:**

1. **Query Timing Analysis:**
```bash
# Use debug to get timing breakdown
debugQuery: true
```

2. **Check Cache Performance:**
    - Review cache hit ratios in **Plugins/Stats**
    - Look for high eviction rates
    - Consider cache size adjustments

3. **Index Analysis:**
    - Check number of segments in **Overview**
    - Consider index optimization
    - Review index size and document count

4. **Query Complexity:**
    - Simplify complex boolean queries
    - Use filter queries for exact matches
    - Consider query restructuring

**Problem: High memory usage**

**Memory Analysis:**

1. **Check Memory Statistics:**
    - Review **Dashboard** memory information
    - Monitor heap and non-heap usage
    - Check garbage collection activity

2. **Cache Analysis:**
    - Review cache sizes in **Plugins/Stats**
    - Consider reducing cache sizes
    - Monitor eviction rates

3. **Index Size:**
    - Check index size in **Overview**
    - Consider field storage optimization
    - Review unnecessary stored fields

### Schema Problems

**Problem: Unknown field errors**

**Schema Troubleshooting:**

1. **Verify Field Definition:**
    - Check **Schema** interface for field existence
    - Verify field name spelling and case
    - Check dynamic field patterns

2. **For Managed Schema:**
```bash
# Add field via API call or admin interface
curl -X POST -H 'Content-type:application/json' \
  --data-binary '{
    "add-field": {
      "name":"new_field_tesim",
      "type":"text_general",
      "stored":true,
      "indexed":true,
      "multiValued":true
    }
  }' \
  http://localhost:8983/solr/myapp_development/schema
```

3. **For Traditional Schema:**
    - Edit `schema.xml` file
    - Add field definition
    - Reload core

4. **Test Field Addition:**
    - Use **Schema** interface to verify field exists
    - Test with sample document

**Problem: Schema reload failures**

**Schema Validation:**

1. **Check Syntax:**
    - Validate XML syntax in schema file
    - Check field type definitions
    - Verify copy field references

2. **Review Error Messages:**
    - Check **Logging** interface for detailed errors
    - Look for specific validation failures
    - Review stack traces

3. **Test Incremental Changes:**
    - Make small schema changes
    - Test each change individually
    - Isolate problematic definitions

### Connection and Core Issues

**Problem: Core not accessible**

**Core Troubleshooting:**

1. **Check Core Status:**
    - Go to **Core Admin**
    - Look for error indicators
    - Review core status information

2. **Core Reload:**
    - Try reloading the core
    - Check for reload errors
    - Monitor log messages

3. **Configuration Validation:**
    - Check `solrconfig.xml` syntax
    - Verify schema file exists and is valid
    - Review directory permissions

4. **Core Recreation:**
```bash
# If necessary, recreate core
./solr delete -c problematic_core
./solr create -c new_core -d existing_config
```

**Problem: Solr not responding**

**System Diagnosis:**

1. **Check Solr Process:**
```bash
# Check if Solr is running
./solr status

# Check port availability
netstat -an | grep 8983
```

2. **Review System Resources:**
    - Check available memory
    - Monitor CPU usage
    - Check disk space

3. **Log Analysis:**
```bash
# Check Solr logs for errors
tail -f solr/server/logs/solr.log

# Check for out of memory errors
grep -i "outofmemory" solr/server/logs/solr.log
```

4. **Restart Procedure:**
```bash
# Graceful restart
./solr stop
./solr start

# Force restart if needed
./solr stop -force
./solr start
```

---
