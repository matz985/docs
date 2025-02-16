﻿# Revisions: Conflict Revisions Configuration

---

{NOTE: }

* RavenDB can create revisions for document conflicts 
  when the conflicts occur and when they are resolved.  

* To manage the creation and purging of revisions for conflicting 
  documents, a **Conflict Revisions Configuration** can be applied 
  using the `ConfigureRevisionsForConflictsOperation` store operation .  
  **This configuration applies to all document collections.**  

* RavenDB applies a conflict revisions configuration by default.  
  You will only need to run the operation yourself if you want to 
  disable the feature or change its characteristics (e.g. to modify 
  the number of conflict revisions kept per document).  

* If you defined [default settings](../../../../document-extensions/revisions/client-api/operations/configure-revisions#section-1) 
  for your [Revisions configuration](../../../../document-extensions/revisions/overview#revisions-configuration), 
  these settings **will override** the conflict revisions configuration.  
   * E.g., if the default configuration settings disable the creation 
     of revisions, no revisions will be created for conflicting documents.  

* Collection-specific configurations **will also override** the conflict revisions 
  configuration, for the collections they are defined for.  
   * E.g., if the document conflicts configuration defines that revisions 
     created for conflicting documents will not be purged, but a collection-specific 
     configuration defines an age limit for revisions, revisions for conflicting 
     documents of this collection that exceed this age will be purged.  

* In this page:  
 * [`ConfigureRevisionsForConflictsOperation`](../../../../document-extensions/revisions/client-api/operations/conflict-revisions-configuration#configurerevisionsforconflictsoperation)  
     * [Storage Considerations](../../../../document-extensions/revisions/client-api/operations/conflict-revisions-configuration#storage-considerations)  
 * [Example](../../../../document-extensions/revisions/client-api/operations/conflict-revisions-configuration#example)  

{NOTE/}

---

{PANEL: `ConfigureRevisionsForConflictsOperation`}

Pass `ConfigureRevisionsForConflictsOperation` the name of the database whose 
conflict revisions you want to manage, and a 
[RevisionsCollectionConfiguration](../../../../document-extensions/revisions/client-api/operations/configure-revisions#section-2) 
object with your settings.  
{CODE:csharp ConfigureRevisionsForConflictsOperation_definition@DocumentExtensions\Revisions\ClientAPI\Operations\ConfigureRevisionsDefinitions.cs /}

| Parameter | Type | Description |
| - | - | - |
| **database** | `string` | The name of the database whose conflict revisions you want to manage |
| **configuration** | [RevisionsCollectionConfiguration](../../../../document-extensions/revisions/client-api/operations/configure-revisions#section-2) | The conflict revisions configuration to apply |

---

### Storage Considerations

Tracking document conflicts and understanding their reasons becomes much easier 
when conflict revisions are created automatically.  
However, if many conflicts occur unexpectedly it may trigger the creation of 
a large number of conflict revisions and increase the database size significantly.  

* Limit the number of conflict revisions kept per document using the 
  `MinimumRevisionsToKeep` and `MinimumRevisionAgeToKeep` 
  [RevisionsCollectionConfiguration](../../../../document-extensions/revisions/client-api/operations/configure-revisions#section-2) 
  properties.  
* Revisions are purged upon [modification of their parent documents](../../../../document-extensions/revisions/overview#revisions-configuration-execution).  
  If you want to purge a large number of revisions at once, you can **cautiously** use 
  [Enforce Configuration](../../../../studio/database/settings/document-revisions#enforce-configuration).  

{PANEL/}

{PANEL: Example}

In this example we create a Conflict Revisions configuration that 
[purges conflict revisions](../../../../document-extensions/revisions/client-api/operations/conflict-revisions-configuration#storage-considerations) 
when their parent documents are deleted or their age exceeds 45 days.  

{CODE-TABS}
{CODE-TAB:csharp:Sync conflict-revisions-configuration_sync@DocumentExtensions\Revisions\ClientAPI\Operations\ConfigureRevisions.cs /}
{CODE-TAB:csharp:Async conflict-revisions-configuration_async@DocumentExtensions\Revisions\ClientAPI\Operations\ConfigureRevisions.cs /}
{CODE-TABS/}

{PANEL/}

## Related Articles

### Client API

* [Operations: Configuring Revisions](../../../../document-extensions/revisions/client-api/operations/configure-revisions)  
* [Revisions: API Overview](../../../../document-extensions/revisions/client-api/overview)  
* [Session: Loading Revisions](../../../../document-extensions/revisions/client-api/session/loading)  
* [Session: Including Revisions](../../../../document-extensions/revisions/client-api/session/including)  
* [Session: Counting Revisions](../../../../document-extensions/revisions/client-api/session/counting)  

### Document Extensions

* [Document Revisions Overview](../../../../document-extensions/revisions/overview)  

### Studio

* [Conflicting Document Defaults](../../../../studio/database/settings/document-revisions#editing-the-conflicting-document-defaults)  
* [Settings: Document Revisions](../../../../studio/database/settings/document-revisions)  
* [Document Extensions: Revisions](../../../../studio/database/document-extensions/revisions)  
