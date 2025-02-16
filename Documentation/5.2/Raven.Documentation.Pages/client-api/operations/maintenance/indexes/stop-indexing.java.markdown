# Pause Indexing Operation

**StopIndexingOperation** is used to pause indexing for the entire database.

Use [StopIndexOperation](../../../../client-api/operations/maintenance/indexes/stop-index) to stop single index.

{NOTE Indexing will be resumed automatically after a server restart or after using [start indexing operation](../../../../client-api/operations/maintenance/indexes/start-indexing)./}

### Syntax

{CODE:java stop_1@ClientApi\Operations\Maintenance\Indexes\PauseIndexing.java /}

### Example

{CODE:java stop_2@ClientApi\Operations\Maintenance\Indexes\PauseIndexing.java /}

## Related Articles

### Indexes

- [What are Indexes](../../../../indexes/what-are-indexes)
- [Creating and Deploying Indexes](../../../../indexes/creating-and-deploying)

### Server

- [Index Administration](../../../../server/administration/index-administration)

### Operations

- [How to Enable Index](../../../../client-api/operations/maintenance/indexes/enable-index)
- [How to Pause Index Until Restart](../../../../client-api/operations/maintenance/indexes/stop-index)
- [How to Resume Indexing](../../../../client-api/operations/maintenance/indexes/start-indexing)
