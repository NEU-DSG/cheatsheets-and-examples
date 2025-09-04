# Solr Installation Guide - Basic Setup with Custom Configuration
Guide to integrating Solr 8.11 indexing functionality for search consistent with DSG project requirements

## Prerequisites
- Solr 8.11 installed as outlined in the **[Apache Solr Reference Guide](https://solr.apache.org/guide/8_11/)**

## Assumptions
- MacOS development environment.
- Project requires use of custom configuration in favor of Solr's default configuration implemented via the ```managed-schema``` file.

---

## Table of Contents
### [Start Solr](#1-start-solr-server)
- Starting the server
- Checking status
### [Create Core](#2-create-solr-core)
- Navigate to core directory
- Verify core.properties
- Remove default configs
- Add custom configuration files
- Edit Configuration Files
### [Restart and Verify](#4-restart-and-verify)
- Restart Solr server
- Verify configuration
   - Command line verification
   - Admin Console access
### [Additional Guidance](additional-guidance)
- [Troubleshooting](#troubleshooting)
- [Solr Admin Console](#solr-admin-console)

---

## Setup
### 1. Start Solr
```bash
cd path/to/solr-version-number
bin/solr start
```
Check status: 
```bash
bin/solr status
```

### 2. Create Solr Core
Create a core:
```bash
bin/solr create -c core-name
```

### 3. Configure Solr Core
Modify core configuration files to include fields and query patterns suitable for your application:

#### Core Configuration Steps
1. **Navigate to core directory**: `cd server/solr/{CORE_NAME}`
2. **Verify core.properties**:
    - Open: `nano core.properties`
    - Ensure name matches the core name
    - Save if changed: `Ctrl+O` → `Y` → `Enter`
    - Exit: `Ctrl+X`
3. **Remove default configs**:
   ```bash
   cd conf
   rm managed-schema.xml schema.xml solrconfig.xml
   ```
4. **Add custom configuration files** (in the `conf` directory):
    - Ensure you're in: `server/solr/{CORE_NAME}/conf/`
    - Create files: `touch solrconfig.xml schema.xml`
    - Review content from the links below for examples of fields defined to optimize Blacklight catalog search and faceting.

#### Core Configuration Files

| Core | solrconfig.xml | schema.xml |
|------|----------------|------------|
| **blacklight-core** | [solrconfig link](https://gist.github.com/candyhazlett/519bac3237faa24dd4d341c0e7fa55a5) | [schema link](https://gist.github.com/candyhazlett/6741bc0df272bdf0f42d0320eac2286c#file-schema-xml) |

#### Quick File Editing Commands
```bash
# For each configuration file:
nano {filename}.xml
# Paste content (Cmd+V)
# Save: Ctrl+O → Y → Enter
# Exit: Ctrl+X
```

### 4. Restart and Verify

1. **Restart Solr**:
   ```bash
   cd path/to/solr-8.11.2
   bin/solr restart
   ```

2. **Verify configuration**:
   ```bash
   # Command line verification
   curl "http://localhost:8983/solr/admin/cores?action=STATUS&wt=json"
   
   # Or Access the Admin Console
   open http://localhost:8983/solr/#/
   ```
   - You should see the core you created in the left sidebar under the 'Cores' menu

---

### Additional Guidance
- **Troubleshooting**
  - Need help? Read the [Troubleshooting Guide](TROUBLESHOOTING.md) for steps to identify and resolve common issues.
- **Solr Admin Console**
  - Read the [Admin Console Guide](ADMIN%20CONSOLE.md) to learn about navigating the Admin Console to manage Solr.


