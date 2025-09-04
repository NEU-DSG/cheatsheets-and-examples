# Solr Field Reference Guide
Solr 8.11 field reference with a focus on optimal full-text search, term search, and faceting capabilities consistent with DSG project requirements.

## Prerequisites
- Solr 8.11 installed as outlined in the **[Apache Solr Reference Guide](https://solr.apache.org/guide/8_11/)**

## Assumptions
- Your project uses a custom configuration, similar to the setup defined in the[Basic Setup Guide](BASIC%20SETUP.md) in favor of Solr's default configuration, implemented through use of the ```managed-schema``` file.
---

## Table of Contents

1. [Field Naming Conventions](#field-naming-conventions)
2. [Core Field Types](#core-field-types)
3. [Full-Text Search Configuration](#full-text-search-configuration)
4. [Term Search Configuration](#term-search-configuration)
5. [Blacklight Faceting Configuration](#blacklight-faceting-configuration)
6. [Complete Schema Examples](#complete-schema-examples)
7. [Search Handler Configuration](#search-handler-configuration)
8. [Performance Optimization](#performance-optimization)
9. [Best Practices](#best-practices)

---

## Field Naming Conventions

### Blacklight Standard Suffixes

#### Multi-Valued Fields
| Suffix | Purpose | Indexed | Stored | Multi-Valued | Use Case |
|--------|---------|---------|--------|--------------|----------|
| `_tesim` | Text, English, Stored, Indexed, Multi-valued | ✓ | ✓ | ✓ | Full-text search + display |
| `_teim` | Text, English, Indexed, Multi-valued | ✓ | ✗ | ✓ | Full-text search only |
| `_sim` | String, Indexed, Multi-valued | ✓ | ✓ | ✓ | Faceting + exact match |
| `_ssim` | String, Stored, Indexed, Multi-valued | ✓ | ✓ | ✓ | Faceting + display |
| `_tim` | Text, Indexed, Multi-valued | ✓ | ✗ | ✓ | Generic text search |
| `_isim` | Integer, Stored, Indexed, Multi-valued | ✓ | ✓ | ✓ | Numeric faceting |
| `_dtsim` | DateTime, Stored, Indexed, Multi-valued | ✓ | ✓ | ✓ | Date faceting |

#### Single-Valued Fields
| Suffix | Purpose | Indexed | Stored | Multi-Valued | Use Case |
|--------|---------|---------|--------|--------------|----------|
| `_ssi` | String, Stored, Indexed, Single-valued | ✓ | ✓ | ✗ | Unique identifiers, exact match + display |
| `_ss` | String, Stored, Single-valued | ✗ | ✓ | ✗ | Display-only values |
| `_si` | String, Indexed, Single-valued | ✓ | ✗ | ✗ | Exact match, no display |
| `_s` | String, Single-valued | ✗ | ✗ | ✗ | Minimal storage (rare) |
| `_tesi` | Text, English, Stored, Indexed, Single-valued | ✓ | ✓ | ✗ | Single title/description for search + display |
| `_tei` | Text, English, Indexed, Single-valued | ✓ | ✗ | ✗ | Single searchable text field |
| `_isi` | Integer, Stored, Indexed, Single-valued | ✓ | ✓ | ✗ | Unique numeric values |
| `_dtsi` | DateTime, Stored, Indexed, Single-valued | ✓ | ✓ | ✗ | Single date values |
| `_bsi` | Boolean, Stored, Indexed, Single-valued | ✓ | ✓ | ✗ | True/false flags |

### Field Naming Examples

#### Multi-Valued Fields
```xml
<!-- Core bibliographic fields -->
<field name="title_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
<field name="author_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
<field name="subject_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>

<!-- Faceting fields -->
<field name="format_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="language_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="publication_year_isim" type="pint" indexed="true" stored="true" multiValued="true"/>

<!-- Search-only fields -->
<field name="fulltext_teim" type="text_general" indexed="true" stored="false" multiValued="true"/>
<field name="searchable_text_teim" type="text_general" indexed="true" stored="false" multiValued="true"/>
```

#### Single-Valued Fields
```xml
<!-- Unique identifiers and primary fields -->
<field name="identifier_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
<field name="isbn_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
<field name="doi_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
<field name="oclc_number_ssi" type="string" indexed="true" stored="true" multiValued="false"/>

<!-- Primary title (main title, not alternative titles) -->
<field name="main_title_tesi" type="text_general" indexed="true" stored="true" multiValued="false"/>
<field name="primary_author_tesi" type="text_general" indexed="true" stored="true" multiValued="false"/>

<!-- Single date values -->
<field name="publication_date_dtsi" type="pdate" indexed="true" stored="true" multiValued="false"/>
<field name="created_date_dtsi" type="pdate" indexed="true" stored="true" multiValued="false"/>
<field name="last_modified_dtsi" type="pdate" indexed="true" stored="true" multiValued="false"/>

<!-- Numeric single values -->
<field name="page_count_isi" type="pint" indexed="true" stored="true" multiValued="false"/>
<field name="edition_number_isi" type="pint" indexed="true" stored="true" multiValued="false"/>
<field name="volume_number_isi" type="pint" indexed="true" stored="true" multiValued="false"/>

<!-- Boolean flags -->
<field name="open_access_bsi" type="boolean" indexed="true" stored="true" multiValued="false"/>
<field name="peer_reviewed_bsi" type="boolean" indexed="true" stored="true" multiValued="false"/>
<field name="digitized_bsi" type="boolean" indexed="true" stored="true" multiValued="false"/>

<!-- Display-only fields -->
<field name="catalog_url_ss" type="string" indexed="false" stored="true" multiValued="false"/>
<field name="thumbnail_url_ss" type="string" indexed="false" stored="true" multiValued="false"/>
<field name="full_citation_ss" type="string" indexed="false" stored="true" multiValued="false"/>

<!-- Sorting fields -->
<field name="title_sort_s" type="string_lowercase" indexed="true" stored="false" multiValued="false"/>
<field name="author_sort_s" type="string_lowercase" indexed="true" stored="false" multiValued="false"/>
<field name="date_sort_dtsi" type="pdate" indexed="true" stored="false" multiValued="false"/>
```

---

## Core Field Types

### 1. Text Field Types for Full-Text Search

#### General Text (English-optimized)
```xml
<fieldType name="text_general" class="solr.TextField" positionIncrementGap="100">
  <analyzer type="index">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
    <filter class="solr.LowerCaseFilterFactory"/>
    <filter class="solr.EnglishPossessiveFilterFactory"/>
    <filter class="solr.PorterStemFilterFactory"/>
  </analyzer>
  <analyzer type="query">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
    <filter class="solr.LowerCaseFilterFactory"/>
    <filter class="solr.EnglishPossessiveFilterFactory"/>
    <filter class="solr.PorterStemFilterFactory"/>
  </analyzer>
</fieldType>
```

#### Advanced Text with Synonyms
```xml
<fieldType name="text_general_syn" class="solr.TextField" positionIncrementGap="100">
  <analyzer type="index">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
    <filter class="solr.LowerCaseFilterFactory"/>
    <filter class="solr.EnglishPossessiveFilterFactory"/>
    <filter class="solr.PorterStemFilterFactory"/>
  </analyzer>
  <analyzer type="query">
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
    <filter class="solr.LowerCaseFilterFactory"/>
    <filter class="solr.SynonymGraphFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
    <filter class="solr.EnglishPossessiveFilterFactory"/>
    <filter class="solr.PorterStemFilterFactory"/>
  </analyzer>
</fieldType>
```

#### Minimal Processing Text (for exact phrase matching)
```xml
<fieldType name="text_ws" class="solr.TextField" positionIncrementGap="100">
  <analyzer>
    <tokenizer class="solr.WhitespaceTokenizerFactory"/>
    <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
</fieldType>
```

### 2. String Types for Faceting and Exact Matching

#### Basic String
```xml
<fieldType name="string" class="solr.StrField" sortMissingLast="true"/>
```

#### Lowercase String for Case-Insensitive Operations
```xml
<fieldType name="string_lowercase" class="solr.TextField" sortMissingLast="true" omitNorms="true">
  <analyzer>
    <tokenizer class="solr.KeywordTokenizerFactory"/>
    <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
</fieldType>
```

### 3. Numeric and Date Types
```xml
<!-- Integer -->
<fieldType name="pint" class="solr.IntPointField" docValues="true"/>

<!-- Float -->
<fieldType name="pfloat" class="solr.FloatPointField" docValues="true"/>

<!-- Date -->
<fieldType name="pdate" class="solr.DatePointField" docValues="true"/>

<!-- Boolean -->
<fieldType name="boolean" class="solr.BoolField" sortMissingLast="true"/>

<!-- Long -->
<fieldType name="plong" class="solr.LongPointField" docValues="true"/>
```

---

## Full-Text Search Configuration

### Essential Fields for Full-Text Search

```xml
<!-- Primary content fields (multi-valued) -->
<field name="title_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
<field name="subtitle_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
<field name="author_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
<field name="description_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
<field name="subject_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
<field name="publisher_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>

<!-- Primary single-valued fields -->
<field name="main_title_tesi" type="text_general" indexed="true" stored="true" multiValued="false"/>
<field name="primary_author_tesi" type="text_general" indexed="true" stored="true" multiValued="false"/>
<field name="primary_description_tesi" type="text_general" indexed="true" stored="true" multiValued="false"/>

<!-- Full-text content -->
<field name="fulltext_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
<field name="abstract_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
<field name="contents_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>

<!-- Searchable text aggregation field -->
<field name="text" type="text_general" indexed="true" stored="false" multiValued="true"/>
```

### Copy Fields for Comprehensive Search
```xml
<!-- Copy all searchable content to a single field -->
<copyField source="title_tesim" dest="text"/>
<copyField source="main_title_tesi" dest="text"/>
<copyField source="subtitle_tesim" dest="text"/>
<copyField source="author_tesim" dest="text"/>
<copyField source="primary_author_tesi" dest="text"/>
<copyField source="description_tesim" dest="text"/>
<copyField source="primary_description_tesi" dest="text"/>
<copyField source="subject_tesim" dest="text"/>
<copyField source="publisher_tesim" dest="text"/>
<copyField source="fulltext_tesim" dest="text"/>
<copyField source="abstract_tesim" dest="text"/>
<copyField source="contents_tesim" dest="text"/>

<!-- Create specific search fields -->
<copyField source="*_tesim" dest="all_text_tesim"/>
<copyField source="*_tesi" dest="all_text_tesim"/>
```

### Search Boost Configuration
```xml
<!-- Request handler with field boosting -->
<requestHandler name="/search" class="solr.SearchHandler">
  <lst name="defaults">
    <str name="defType">edismax</str>
    <str name="qf">
      main_title_tesi^10.0
      title_tesim^5.0
      subtitle_tesim^3.0
      primary_author_tesi^4.0
      author_tesim^2.0
      subject_tesim^1.5
      description_tesim^1.0
      primary_description_tesi^1.2
      fulltext_tesim^0.5
      text^0.1
    </str>
    <str name="pf">
      main_title_tesi^100.0
      title_tesim^50.0
      subtitle_tesim^30.0
      primary_author_tesi^40.0
      author_tesim^20.0
    </str>
    <str name="rows">20</str>
    <str name="fl">*,score</str>
    <str name="mm">2&lt;-1 5&lt;-2 6&lt;90%</str>
  </lst>
</requestHandler>
```

---

## Term Search Configuration

### Exact Term Matching Fields
```xml
<!-- Multi-valued exact matching -->
<field name="author_exact_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="subject_exact_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="title_exact_sim" type="string" indexed="true" stored="true" multiValued="true"/>

<!-- Single-valued exact matching -->
<field name="primary_author_exact_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
<field name="main_title_exact_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
<field name="isbn_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
<field name="doi_ssi" type="string" indexed="true" stored="true" multiValued="false"/>

<!-- Lowercase versions for case-insensitive exact matching -->
<field name="author_lower_sim" type="string_lowercase" indexed="true" stored="false" multiValued="true"/>
<field name="subject_lower_sim" type="string_lowercase" indexed="true" stored="false" multiValued="true"/>
<field name="primary_author_lower_ssi" type="string_lowercase" indexed="true" stored="false" multiValued="false"/>
```

### Copy Fields for Term Search
```xml
<!-- Create exact match versions -->
<copyField source="author_tesim" dest="author_exact_sim"/>
<copyField source="subject_tesim" dest="subject_exact_sim"/>
<copyField source="title_tesim" dest="title_exact_sim"/>
<copyField source="primary_author_tesi" dest="primary_author_exact_ssi"/>
<copyField source="main_title_tesi" dest="main_title_exact_ssi"/>

<!-- Create lowercase versions -->
<copyField source="author_tesim" dest="author_lower_sim"/>
<copyField source="subject_tesim" dest="subject_lower_sim"/>
<copyField source="primary_author_tesi" dest="primary_author_lower_ssi"/>
```

### Auto-suggest/Type-ahead Configuration
```xml
<!-- Suggest field type -->
<fieldType name="text_suggest" class="solr.TextField" positionIncrementGap="100">
  <analyzer>
    <tokenizer class="solr.StandardTokenizerFactory"/>
    <filter class="solr.LowerCaseFilterFactory"/>
    <filter class="solr.ASCIIFoldingFilterFactory"/>
  </analyzer>
</fieldType>

<!-- Suggest fields -->
<field name="suggest_title" type="text_suggest" indexed="true" stored="false" multiValued="true"/>
<field name="suggest_author" type="text_suggest" indexed="true" stored="false" multiValued="true"/>
<field name="suggest_subject" type="text_suggest" indexed="true" stored="false" multiValued="true"/>

<!-- Copy to suggest fields -->
<copyField source="title_tesim" dest="suggest_title"/>
<copyField source="main_title_tesi" dest="suggest_title"/>
<copyField source="author_tesim" dest="suggest_author"/>
<copyField source="primary_author_tesi" dest="suggest_author"/>
<copyField source="subject_tesim" dest="suggest_subject"/>
```

---

## Blacklight Faceting Configuration

### Standard Facet Fields

```xml
<!-- Format/Type facets (multi-valued) -->
<field name="format_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="resource_type_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="genre_sim" type="string" indexed="true" stored="true" multiValued="true"/>

<!-- Primary format (single-valued for main format) -->
<field name="primary_format_ssi" type="string" indexed="true" stored="true" multiValued="false"/>

<!-- Creator/Contributor facets -->
<field name="author_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="creator_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="contributor_sim" type="string" indexed="true" stored="true" multiValued="true"/>

<!-- Primary creator (single-valued) -->
<field name="primary_creator_ssi" type="string" indexed="true" stored="true" multiValued="false"/>

<!-- Subject/Topic facets -->
<field name="subject_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="topic_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="discipline_sim" type="string" indexed="true" stored="true" multiValued="true"/>

<!-- Publication facets -->
<field name="publisher_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="publication_place_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="publication_year_isim" type="pint" indexed="true" stored="true" multiValued="true"/>
<field name="publication_decade_sim" type="string" indexed="true" stored="true" multiValued="true"/>

<!-- Single publication fields -->
<field name="primary_publisher_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
<field name="publication_year_isi" type="pint" indexed="true" stored="true" multiValued="false"/>

<!-- Language and geographic facets -->
<field name="language_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="geographic_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="region_sim" type="string" indexed="true" stored="true" multiValued="true"/>

<!-- Primary language (single-valued) -->
<field name="primary_language_ssi" type="string" indexed="true" stored="true" multiValued="false"/>

<!-- Access and availability facets -->
<field name="access_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="availability_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="collection_sim" type="string" indexed="true" stored="true" multiValued="true"/>

<!-- Boolean availability flags -->
<field name="open_access_bsi" type="boolean" indexed="true" stored="true" multiValued="false"/>
<field name="available_online_bsi" type="boolean" indexed="true" stored="true" multiValued="false"/>
<field name="peer_reviewed_bsi" type="boolean" indexed="true" stored="true" multiValued="false"/>
```

### Hierarchical Faceting
```xml
<!-- For hierarchical subject browsing -->
<field name="subject_hierarchy_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<field name="geographic_hierarchy_sim" type="string" indexed="true" stored="true" multiValued="true"/>

<!-- Example hierarchical values:
     "Science"
     "Science/Biology"
     "Science/Biology/Genetics"
-->
```

### Date Range Faceting
```xml
<!-- Date fields for range faceting (single-valued) -->
<field name="publication_date_dtsi" type="pdate" indexed="true" stored="true" multiValued="false"/>
<field name="created_date_dtsi" type="pdate" indexed="true" stored="true" multiValued="false"/>
<field name="modified_date_dtsi" type="pdate" indexed="true" stored="true" multiValued="false"/>

<!-- Date fields for range faceting (multi-valued) -->
<field name="publication_date_dtsim" type="pdate" indexed="true" stored="true" multiValued="true"/>
<field name="created_date_dtsim" type="pdate" indexed="true" stored="true" multiValued="true"/>
<field name="modified_date_dtsim" type="pdate" indexed="true" stored="true" multiValued="true"/>

<!-- Date range fields -->
<field name="date_range_sim" type="string" indexed="true" stored="true" multiValued="true"/>
<!-- Example values: "1900-1999", "2000-2009", "2010-2019" -->
```

---

## Complete Schema Examples

### Minimal Blacklight Schema with Single-Valued Fields
```xml
<?xml version="1.0" encoding="UTF-8"?>
<schema name="blacklight-core" version="1.6">

  <!-- Field Types -->
  <fieldType name="string" class="solr.StrField" sortMissingLast="true"/>
  <fieldType name="pint" class="solr.IntPointField" docValues="true"/>
  <fieldType name="pdate" class="solr.DatePointField" docValues="true"/>
  <fieldType name="plong" class="solr.LongPointField" docValues="true"/>
  <fieldType name="boolean" class="solr.BoolField" sortMissingLast="true"/>
  
  <fieldType name="text_general" class="solr.TextField" positionIncrementGap="100">
    <analyzer type="index">
      <tokenizer class="solr.StandardTokenizerFactory"/>
      <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
      <filter class="solr.LowerCaseFilterFactory"/>
      <filter class="solr.EnglishPossessiveFilterFactory"/>
      <filter class="solr.PorterStemFilterFactory"/>
    </analyzer>
    <analyzer type="query">
      <tokenizer class="solr.StandardTokenizerFactory"/>
      <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
      <filter class="solr.LowerCaseFilterFactory"/>
      <filter class="solr.EnglishPossessiveFilterFactory"/>
      <filter class="solr.PorterStemFilterFactory"/>
    </analyzer>
  </fieldType>

  <fieldType name="string_lowercase" class="solr.TextField" sortMissingLast="true" omitNorms="true">
    <analyzer>
      <tokenizer class="solr.KeywordTokenizerFactory"/>
      <filter class="solr.LowerCaseFilterFactory"/>
    </analyzer>
  </fieldType>

  <!-- Required Fields -->
  <field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false"/>
  <field name="_version_" type="plong" indexed="false" stored="false" docValues="false"/>
  <field name="timestamp" type="pdate" indexed="true" stored="true" default="NOW" multiValued="false"/>

  <!-- Primary Display Fields (single-valued) -->
  <field name="main_title_tesi" type="text_general" indexed="true" stored="true" multiValued="false"/>
  <field name="primary_author_tesi" type="text_general" indexed="true" stored="true" multiValued="false"/>
  <field name="primary_description_tesi" type="text_general" indexed="true" stored="true" multiValued="false"/>

  <!-- Additional Display Fields (multi-valued) -->
  <field name="title_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
  <field name="author_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
  <field name="description_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>

  <!-- Identifiers (single-valued) -->
  <field name="isbn_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
  <field name="doi_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
  <field name="oclc_number_ssi" type="string" indexed="true" stored="true" multiValued="false"/>

  <!-- Primary Facet Fields (single-valued) -->
  <field name="primary_format_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
  <field name="primary_language_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
  <field name="publication_year_isi" type="pint" indexed="true" stored="true" multiValued="false"/>

  <!-- Multi-valued Facet Fields -->
  <field name="format_sim" type="string" indexed="true" stored="true" multiValued="true"/>
  <field name="author_sim" type="string" indexed="true" stored="true" multiValued="true"/>
  <field name="subject_sim" type="string" indexed="true" stored="true" multiValued="true"/>
  <field name="publication_year_isim" type="pint" indexed="true" stored="true" multiValued="true"/>
  <field name="language_sim" type="string" indexed="true" stored="true" multiValued="true"/>

  <!-- Boolean flags -->
  <field name="open_access_bsi" type="boolean" indexed="true" stored="true" multiValued="false"/>
  <field name="peer_reviewed_bsi" type="boolean" indexed="true" stored="true" multiValued="false"/>

  <!-- Date fields -->
  <field name="publication_date_dtsi" type="pdate" indexed="true" stored="true" multiValued="false"/>
  <field name="created_date_dtsi" type="pdate" indexed="true" stored="true" multiValued="false"/>

  <!-- Search Field -->
  <field name="text" type="text_general" indexed="true" stored="false" multiValued="true"/>

  <!-- Sorting fields -->
  <field name="title_sort_s" type="string_lowercase" indexed="true" stored="false" multiValued="false"/>
  <field name="author_sort_s" type="string_lowercase" indexed="true" stored="false" multiValued="false"/>

  <!-- Copy Fields -->
  <copyField source="main_title_tesi" dest="text"/>
  <copyField source="title_tesim" dest="text"/>
  <copyField source="primary_author_tesi" dest="text"/>
  <copyField source="author_tesim" dest="text"/>
  <copyField source="primary_description_tesi" dest="text"/>
  <copyField source="description_tesim" dest="text"/>
  
  <!-- Copy to facet fields -->
  <copyField source="author_tesim" dest="author_sim"/>
  <copyField source="primary_author_tesi" dest="author_sim"/>

  <!-- Copy to sort fields -->
  <copyField source="main_title_tesi" dest="title_sort_s" maxChars="256"/>
  <copyField source="primary_author_tesi" dest="author_sort_s" maxChars="256"/>

  <!-- Unique Key -->
  <uniqueKey>id</uniqueKey>

</schema>
```

### Advanced Blacklight Schema with All Features
```xml
<?xml version="1.0" encoding="UTF-8"?>
<schema name="blacklight-advanced" version="1.6">

  <!-- Field Types -->
  <fieldType name="string" class="solr.StrField" sortMissingLast="true"/>
  <fieldType name="pint" class="solr.IntPointField" docValues="true"/>
  <fieldType name="plong" class="solr.LongPointField" docValues="true"/>
  <fieldType name="pdate" class="solr.DatePointField" docValues="true"/>
  <fieldType name="boolean" class="solr.BoolField" sortMissingLast="true"/>
  
  <fieldType name="text_general" class="solr.TextField" positionIncrementGap="100">
    <analyzer type="index">
      <tokenizer class="solr.StandardTokenizerFactory"/>
      <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
      <filter class="solr.LowerCaseFilterFactory"/>
      <filter class="solr.EnglishPossessiveFilterFactory"/>
      <filter class="solr.PorterStemFilterFactory"/>
    </analyzer>
    <analyzer type="query">
      <tokenizer class="solr.StandardTokenizerFactory"/>
      <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt"/>
      <filter class="solr.LowerCaseFilterFactory"/>
      <filter class="solr.SynonymGraphFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
      <filter class="solr.EnglishPossessiveFilterFactory"/>
      <filter class="solr.PorterStemFilterFactory"/>
    </analyzer>
  </fieldType>

  <fieldType name="string_lowercase" class="solr.TextField" sortMissingLast="true" omitNorms="true">
    <analyzer>
      <tokenizer class="solr.KeywordTokenizerFactory"/>
      <filter class="solr.LowerCaseFilterFactory"/>
    </analyzer>
  </fieldType>

  <!-- Required System Fields -->
  <field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false"/>
  <field name="_version_" type="plong" indexed="false" stored="false" docValues="false"/>
  <field name="timestamp" type="pdate" indexed="true" stored="true" default="NOW" multiValued="false"/>

  <!-- Primary Title Fields -->
  <field name="main_title_tesi" type="text_general" indexed="true" stored="true" multiValued="false"/>
  <field name="title_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
  <field name="subtitle_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
  <field name="title_alternative_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
  <field name="title_sort_s" type="string_lowercase" indexed="true" stored="false" multiValued="false"/>

  <!-- Primary Creator/Contributor Fields -->
  <field name="primary_author_tesi" type="text_general" indexed="true" stored="true" multiValued="false"/>
  <field name="author_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
  <field name="creator_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
  <field name="contributor_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
  <field name="editor_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>

  <!-- Content Fields -->
  <field name="primary_description_tesi" type="text_general" indexed="true" stored="true" multiValued="false"/>
  <field name="description_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
  <field name="abstract_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
  <field name="contents_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
  <field name="fulltext_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>

  <!-- Subject Fields -->
  <field name="subject_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
  <field name="keyword_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
  <field name="topic_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>

  <!-- Publication Fields -->
  <field name="publisher_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
  <field name="publication_place_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>
  <field name="publication_date_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>

  <!-- Single-valued Publication Fields -->
  <field name="primary_publisher_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
  <field name="publication_year_isi" type="pint" indexed="true" stored="true" multiValued="false"/>
  <field name="publication_date_dtsi" type="pdate" indexed="true" stored="true" multiValued="false"/>

  <!-- Identifiers (single-valued) -->
  <field name="isbn_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
  <field name="issn_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
  <field name="doi_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
  <field name="oclc_number_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
  <field name="local_identifier_ssi" type="string" indexed="true" stored="true" multiValued="false"/>

  <!-- Multi-valued Facet Fields -->
  <field name="format_sim" type="string" indexed="true" stored="true" multiValued="true"/>
  <field name="resource_type_sim" type="string" indexed="true" stored="true" multiValued="true"/>
  <field name="genre_sim" type="string" indexed="true" stored="true" multiValued="true"/>
  <field name="author_sim" type="string" indexed="true" stored="true" multiValued="true"/>
  <field name="creator_sim" type="string" indexed="true" stored="true" multiValued="true"/>
  <field name="subject_sim" type="string" indexed="true" stored="true" multiValued="true"/>
  <field name="topic_sim" type="string" indexed="true" stored="true" multiValued="true"/>
  <field name="publisher_sim" type="string" indexed="true" stored="true" multiValued="true"/>
  <field name="publication_year_isim" type="pint" indexed="true" stored="true" multiValued="true"/>
  <field name="publication_decade_sim" type="string" indexed="true" stored="true" multiValued="true"/>
  <field name="language_sim" type="string" indexed="true" stored="true" multiValued="true"/>
  <field name="geographic_sim" type="string" indexed="true" stored="true" multiValued="true"/>
  <field name="collection_sim" type="string" indexed="true" stored="true" multiValued="true"/>

  <!-- Single-valued Facet Fields -->
  <field name="primary_format_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
  <field name="primary_creator_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
  <field name="primary_language_ssi" type="string" indexed="true" stored="true" multiValued="false"/>
  <field name="primary_collection_ssi" type="string" indexed="true" stored="true" multiValued="false"/>

  <!-- Boolean Fields -->
  <field name="open_access_bsi" type="boolean" indexed="true" stored="true" multiValued="false"/>
  <field name="peer_reviewed_bsi" type="boolean" indexed="true" stored="true" multiValued="false"/>
  <field name="digitized_bsi" type="boolean" indexed="true" stored="true" multiValued="false"/>
  <field name="available_online_bsi" type="boolean" indexed="true" stored="true" multiValued="false"/>

  <!-- Numeric Fields -->
  <field name="page_count_isi" type="pint" indexed="true" stored="true" multiValued="false"/>
  <field name="volume_number_isi" type="pint" indexed="true" stored="true" multiValued="false"/>
  <field name="issue_number_isi" type="pint" indexed="true" stored="true" multiValued="false"/>

  <!-- Date Fields -->
  <field name="publication_date_dtsim" type="pdate" indexed="true" stored="true" multiValued="true"/>
  <field name="created_date_dtsi" type="pdate" indexed="true" stored="true" multiValued="false"/>
  <field name="created_date_dtsim" type="pdate" indexed="true" stored="true" multiValued="true"/>
  <field name="modified_date_dtsi" type="pdate" indexed="true" stored="true" multiValued="false"/>
  <field name="modified_date_dtsim" type="pdate" indexed="true" stored="true" multiValued="true"/>

  <!-- Display-only Fields -->
  <field name="catalog_url_ss" type="string" indexed="false" stored="true" multiValued="false"/>
  <field name="thumbnail_url_ss" type="string" indexed="false" stored="true" multiValued="false"/>
  <field name="full_text_url_ss" type="string" indexed="false" stored="true" multiValued="false"/>
  <field name="citation_ss" type="string" indexed="false" stored="true" multiValued="false"/>

  <!-- Search Fields -->
  <field name="text" type="text_general" indexed="true" stored="false" multiValued="true"/>
  <field name="all_text_tesim" type="text_general" indexed="true" stored="false" multiValued="true"/>

  <!-- Sorting Fields -->
  <field name="author_sort_s" type="string_lowercase" indexed="true" stored="false" multiValued="false"/>
  <field name="date_sort_dtsi" type="pdate" indexed="true" stored="false" multiValued="false"/>

  <!-- Copy Fields for Search -->
  <copyField source="*_tesim" dest="text"/>
  <copyField source="*_tesi" dest="text"/>
  <copyField source="*_tesim" dest="all_text_tesim"/>
  <copyField source="*_tesi" dest="all_text_tesim"/>
  
  <!-- Copy Fields for Faceting -->
  <copyField source="author_tesim" dest="author_sim"/>
  <copyField source="primary_author_tesi" dest="author_sim"/>
  <copyField source="creator_tesim" dest="creator_sim"/>
  <copyField source="subject_tesim" dest="subject_sim"/>
  <copyField source="publisher_tesim" dest="publisher_sim"/>

  <!-- Copy Fields for Single-valued Faceting -->
  <copyField source="primary_author_tesi" dest="primary_creator_ssi"/>

  <!-- Copy Fields for Sorting -->
  <copyField source="main_title_tesi" dest="title_sort_s" maxChars="256"/>
  <copyField source="primary_author_tesi" dest="author_sort_s" maxChars="256"/>
  <copyField source="publication_date_dtsi" dest="date_sort_dtsi"/>

  <!-- Unique Key -->
  <uniqueKey>id</uniqueKey>

</schema>
```

---

## Search Handler Configuration

### Basic Search Handler with Single-Valued Field Support
```xml
<requestHandler name="/search" class="solr.SearchHandler">
  <lst name="defaults">
    <str name="defType">edismax</str>
    <str name="echoParams">explicit</str>
    <str name="q.alt">*:*</str>
    <str name="mm">2&lt;-1 5&lt;-2 6&lt;90%</str>
    <int name="qs">1</int>
    <int name="ps">2</int>
    <float name="tie">0.01</float>
    
    <!-- Field weights for searching (prioritize single-valued primary fields) -->
    <str name="qf">
      main_title_tesi^10.0
      title_tesim^5.0
      subtitle_tesim^3.0
      primary_author_tesi^6.0
      author_tesim^2.0
      subject_tesim^1.5
      primary_description_tesi^2.0
      description_tesim^1.0
      contents_tesim^0.8
      fulltext_tesim^0.5
      text^0.1
    </str>
    
    <!-- Phrase boost -->
    <str name="pf">
      main_title_tesi^100.0
      title_tesim^50.0
      primary_author_tesi^60.0
      author_tesim^20.0
      subject_tesim^15.0
    </str>
    
    <!-- Default parameters -->
    <str name="fl">*,score</str>
    <str name="wt">json</str>
    <int name="rows">20</int>
    
    <!-- Faceting -->
    <str name="facet">true</str>
    <str name="facet.mincount">1</str>
    <str name="facet.limit">20</str>
    <str name="facet.field">primary_format_ssi</str>
    <str name="facet.field">format_sim</str>
    <str name="facet.field">author_sim</str>
    <str name="facet.field">subject_sim</str>
    <str name="facet.field">publication_year_isi</str>
    <str name="facet.field">publication_year_isim</str>
    <str name="facet.field">primary_language_ssi</str>
    <str name="facet.field">language_sim</str>
    <str name="facet.field">open_access_bsi</str>
    <str name="facet.field">peer_reviewed_bsi</str>
    
    <!-- Highlighting -->
    <str name="hl">true</str>
    <str name="hl.fl">main_title_tesi,title_tesim,primary_author_tesi,author_tesim,primary_description_tesi,description_tesim</str>
    <str name="hl.simple.pre">&lt;mark&gt;</str>
    <str name="hl.simple.post">&lt;/mark&gt;</str>
    <str name="hl.fragsize">250</str>
    <str name="hl.snippets">3</str>
  </lst>
</requestHandler>
```

### Advanced Search Handler with Single/Multi-valued Field Support
```xml
<requestHandler name="/search" class="solr.SearchHandler">
  <lst name="defaults">
    <str name="defType">edismax</str>
    <str name="echoParams">explicit</str>
    <str name="q.alt">*:*</str>
    <str name="mm">2&lt;-1 5&lt;-2 6&lt;90%</str>
    <int name="qs">1</int>
    <int name="ps">2</int>
    <float name="tie">0.01</float>
    
    <!-- Field weights (boost primary single-valued fields higher) -->
    <str name="qf">
      main_title_tesi^10.0
      title_tesim^5.0
      subtitle_tesim^3.0
      primary_author_tesi^8.0
      author_tesim^2.0
      subject_tesim^1.5
      primary_description_tesi^2.5
      description_tesim^1.0
      contents_tesim^0.8
      fulltext_tesim^0.5
      text^0.1
    </str>
    
    <!-- Phrase boost -->
    <str name="pf">main_title_tesi^100.0 title_tesim^50.0 primary_author_tesi^80.0 author_tesim^20.0</str>
    <str name="pf2">main_title_tesi^50.0 title_tesim^25.0 primary_author_tesi^40.0 author_tesim^10.0</str>
    <str name="pf3">main_title_tesi^25.0 title_tesim^15.0 primary_author_tesi^20.0 author_tesim^5.0</str>
    
    <!-- Boost queries for popular formats and recent items -->
    <str name="bq">primary_format_ssi:"Book"^2.0 open_access_bsi:true^1.5 peer_reviewed_bsi:true^1.2</str>
    
    <!-- Default parameters -->
    <str name="fl">*,score</str>
    <str name="wt">json</str>
    <int name="rows">20</int>
    
    <!-- Sort options -->
    <str name="sort">score desc</str>
    
    <!-- Spellcheck -->
    <str name="spellcheck">true</str>
    <str name="spellcheck.dictionary">default</str>
    <str name="spellcheck.onlyMorePopular">false</str>
    <str name="spellcheck.extendedResults">false</str>
    <str name="spellcheck.count">10</str>
    <str name="spellcheck.alternativeTermCount">5</str>
    <str name="spellcheck.maxResultsForSuggest">5</str>
    <str name="spellcheck.collate">true</str>
    <str name="spellcheck.collateExtendedResults">true</str>
    <str name="spellcheck.maxCollationTries">10</str>
    <str name="spellcheck.maxCollations">5</str>
  </lst>
  
  <arr name="last-components">
    <str>spellcheck</str>
  </arr>
</requestHandler>
```

---

## Performance Optimization

### Index-Time Optimizations for Single-Valued Fields

#### Field Configuration
```xml
<!-- Optimize single-valued display fields -->
<field name="main_title_tesi" type="text_general" indexed="true" stored="true" multiValued="false"/>
<field name="primary_author_tesi" type="text_general" indexed="true" stored="true" multiValued="false"/>

<!-- Single-valued search-only fields -->
<field name="main_title_search_only" type="text_general" indexed="true" stored="false" multiValued="false"/>

<!-- Use docValues for single-valued faceting -->
<field name="primary_format_ssi" type="string" indexed="true" stored="true" docValues="true" multiValued="false"/>
<field name="publication_year_isi" type="pint" indexed="true" stored="true" docValues="true" multiValued="false"/>

<!-- Boolean fields are naturally single-valued and efficient -->
<field name="open_access_bsi" type="boolean" indexed="true" stored="true" docValues="true" multiValued="false"/>

<!-- Disable norms for identifier fields -->
<fieldType name="string_id" class="solr.StrField" sortMissingLast="true" omitNorms="true"/>
<field name="isbn_ssi" type="string_id" indexed="true" stored="true" multiValued="false"/>
```

#### Copy Field Optimization with Single-Valued Sources
```xml
<!-- Copy from single-valued to multi-valued aggregation -->
<copyField source="main_title_tesi" dest="text"/>
<copyField source="primary_author_tesi" dest="text"/>

<!-- Limit copy field size for large single-valued text fields -->
<copyField source="primary_description_tesi" dest="text" maxChars="5000"/>

<!-- Create sort fields from single-valued sources -->
<copyField source="main_title_tesi" dest="title_sort_s" maxChars="256"/>
<copyField source="primary_author_tesi" dest="author_sort_s" maxChars="256"/>
```

### Query-Time Optimizations

#### Cache Configuration in solrconfig.xml
```xml
<!-- Document cache -->
<documentCache class="solr.search.LRUCache" 
               size="512" 
               initialSize="512" 
               autowarmCount="0"/>

<!-- Filter cache -->
<filterCache class="solr.search.FastLRUCache"
             size="512"
             initialSize="512"
             autowarmCount="128"/>

<!-- Query result cache -->
<queryResultCache class="solr.search.LRUCache"
                  size="512"
                  initialSize="512"
                  autowarmCount="32"/>

<!-- Field value cache -->
<fieldValueCache class="solr.search.FastLRUCache"
                 size="512"
                 autowarmCount="128"
                 showItems="32"/>
```

---

## Best Practices

### 1. Single vs Multi-Valued Field Selection

#### When to Use Single-Valued Fields (`_ssi`, `_tesi`, etc.)
- **Unique identifiers**: ISBN, DOI, OCLC numbers, local IDs
- **Primary display values**: Main title, primary author, primary format
- **Boolean flags**: Open access, peer reviewed, digitized status
- **Singular dates**: Publication date, creation date, last modified
- **Sorting fields**: Title sort, author sort, date sort
- **Primary classification**: Main subject category, primary language

#### When to Use Multi-Valued Fields (`_sim`, `_tesim`, etc.)
- **Alternative values**: Alternative titles, multiple authors, multiple subjects
- **Faceting with multiple options**: All formats, all languages, all topics
- **Search variations**: Multiple forms of names, synonymous terms
- **Historical data**: Multiple publication dates, version dates

### 2. Field Design Principles

#### Hybrid Approach (Recommended)
```xml
<!-- Primary single-valued field for display and primary search -->
<field name="main_title_tesi" type="text_general" indexed="true" stored="true" multiValued="false"/>

<!-- Multi-valued field for all title variations -->
<field name="title_tesim" type="text_general" indexed="true" stored="true" multiValued="true"/>

<!-- Single-valued facet for primary format -->
<field name="primary_format_ssi" type="string" indexed="true" stored="true" multiValued="false"/>

<!-- Multi-valued facet for all applicable formats -->
<field name="format_sim" type="string" indexed="true" stored="true" multiValued="true"/>

<!-- Boolean for clear yes/no attributes -->
<field name="open_access_bsi" type="boolean" indexed="true" stored="true" multiValued="false"/>
```

### 3. Indexing Strategies

#### Document Structure Example
```xml
<!-- Example document with mixed single/multi-valued fields -->
{
  "id": "item_12345",
  "main_title_tesi": "Complete Guide to Apache Solr",
  "title_tesim": [
    "Complete Guide to Apache Solr", 
    "The Complete Guide to Apache Solr Search Platform"
  ],
  "primary_author_tesi": "John Smith",
  "author_tesim": ["John Smith", "Jane Doe"],
  "isbn_ssi": "978-1234567890",
  "primary_format_ssi": "Book",
  "format_sim": ["Book", "Digital"],
  "publication_year_isi": 2023,
  "publication_year_isim": [2023],
  "open_access_bsi": true,
  "peer_reviewed_bsi": false,
  "publication_date_dtsi": "2023-06-15T00:00:00Z",
  "language_sim": ["English"],
  "primary_language_ssi": "English",
  "subject_sim": ["Information Retrieval", "Search Technology", "Apache Solr"]
}
```

### 4. Search Interface Design

#### Blacklight Configuration Examples with Single-Valued Fields
```ruby
# config/blacklight.rb - Display configuration
config.add_show_field 'main_title_tesi', label: 'Title'
config.add_show_field 'primary_author_tesi', label: 'Author'
config.add_show_field 'isbn_ssi', label: 'ISBN'
config.add_show_field 'primary_format_ssi', label: 'Format'
config.add_show_field 'publication_year_isi', label: 'Publication Year'

# Index/search results configuration
config.add_index_field 'main_title_tesi', label: 'Title'
config.add_index_field 'primary_author_tesi', label: 'Author'
config.add_index_field 'primary_format_ssi', label: 'Format'

# Facet configuration mixing single and multi-valued
config.add_facet_field 'primary_format_ssi', label: 'Primary Format'
config.add_facet_field 'format_sim', label: 'All Formats'
config.add_facet_field 'author_sim', label: 'Authors', limit: 10
config.add_facet_field 'publication_year_isi', label: 'Publication Year'
config.add_facet_field 'open_access_bsi', label: 'Open Access', 
                       query: {
                         'yes' => { fq: 'open_access_bsi:true' },
                         'no' => { fq: 'open_access_bsi:false' }
                       }

# Search field configuration
config.add_search_field 'all_fields', label: 'All Fields' do |field|
  field.include_in_simple_select = true
  field.default = true
  field.solr_parameters = {
    qf: 'main_title_tesi^10.0 title_tesim^5.0 primary_author_tesi^8.0 author_tesim^2.0 subject_tesim^1.5 text^0.1',
    pf: 'main_title_tesi^100.0 title_tesim^50.0 primary_author_tesi^80.0 author_tesim^20.0'
  }
end

# Sort configuration
config.add_sort_field 'score desc', label: 'Relevance'
config.add_sort_field 'title_sort_s asc', label: 'Title A-Z'
config.add_sort_field 'title_sort_s desc', label: 'Title Z-A'
config.add_sort_field 'author_sort_s asc', label: 'Author A-Z'
config.add_sort_field 'publication_year_isi desc', label: 'Publication Year (newest first)'
config.add_sort_field 'publication_year_isi asc', label: 'Publication Year (oldest first)'
```

### 5. Monitoring and Maintenance

#### Field Usage Patterns to Monitor
- **Single-valued field cardinality**: Ensure uniqueness where expected
- **Boolean field distribution**: Monitor true/false ratios
- **Single vs multi-valued performance**: Compare query performance
- **Sort field effectiveness**: Monitor sort field usage and performance

#### Regular Maintenance Tasks
- Validate single-valued field constraints
- Monitor boolean field accuracy
- Review identifier field uniqueness
- Check date field formatting consistency
- Update sort field indexes when primary fields change

---

## Common Query Patterns with Single-Valued Fields

### Exact Matching Queries
```bash
# Search by exact identifier
q=isbn_ssi:9781234567890

# Search by primary format
q=primary_format_ssi:"Book"

# Boolean field queries
q=open_access_bsi:true

# Primary author exact match
q=primary_author_tesi:"John Smith"

# Date range on single-valued field
q=publication_year_isi:[2020 TO 2023]
```

### Mixed Single/Multi-valued Queries
```bash
# Search primary title, facet on multiple formats
q=main_title_tesi:"Apache Solr"&fq=format_sim:("Book" OR "eBook")

# Search all authors, filter by primary format
q=author_tesim:"John Smith"&fq=primary_format_ssi:"Book"

# Boolean filter with multi-valued facet
q=*:*&fq=open_access_bsi:true&fq=subject_sim:"Information Science"
```

### Sorting with Single-Valued Fields
```bash
# Sort by single-valued publication year
q=*:*&sort=publication_year_isi desc

# Sort by title (using single-valued sort field)
q=*:*&sort=title_sort_s asc

# Multiple sort criteria
q=*:*&sort=publication_year_isi desc,title_sort_s asc
```

### Faceting Examples
```bash
# Facet on boolean fields
facet.field=open_access_bsi&facet.field=peer_reviewed_bsi

# Mix single and multi-valued faceting
facet.field=primary_format_ssi&facet.field=format_sim&facet.field=author_sim

# Range faceting on single-valued numeric field
facet.range=publication_year_isi&facet.range.start=1900&facet.range.end=2025&facet.range.gap=10
```
