# Compare Exchange Overview 
---

{NOTE: }

* Compare Exchange items are __key/value pairs__ where the key is unique across your database. 

* Compare-exchange operations require cluster consensus to ensure consistency across all nodes.  
  Once a consensus is reached, the compare-exchange items are distributed through the Raft algorithm to all nodes in the database group.

* Compare-exchange items can be used to coordinate work between sessions that are trying to modify a shared resource (such as a document) at the same time.

* Compare-exchange items are [not replicated externally](../../../client-api/operations/compare-exchange/overview#why-compare-exchange-items-are-not-replicated-to-external-databases) to other databases.

* In this page:  
* [What Compare Exchange Items Are](../../../client-api/operations/compare-exchange/overview#what-compare-exchange-items-are)  
* [Creating and Managing Compare-Exchange Items](../../../client-api/operations/compare-exchange/overview#creating-and-managing-compare-exchange-items)  
* [Why Compare-Exchange Items are Not Replicated to External Databases](../../../client-api/operations/compare-exchange/overview#why-compare-exchange-items-are-not-replicated-to-external-databases)  
* [Example I - Email Address Reservation](../../../client-api/operations/compare-exchange/overview#example-i---email-address-reservation)  
* [Example II - Reserve a Shared Resource](../../../client-api/operations/compare-exchange/overview#example-ii---reserve-a-shared-resource)  
* [Example III - Ensuring Unique Values without Using Compare Exchange](../../../client-api/operations/compare-exchange/overview#example-iii---ensuring-unique-values-without-using-compare-exchange)  

{NOTE/}

---

{PANEL: What Compare Exchange Items Are}

Compare Exchange items are key/value pairs where the key servers a unique value across your database.

* Each compare-exchange item contains: 
  * **A key** - A unique string across the cluster.  
  * **A value** - Can be numbers, strings, arrays, or objects.  
    Any value that can be represented as JSON is valid.
  * **Metadata** - Data that is associated with the item's value.  
    Can be numbers, strings, arrays, or objects.  
    Any value that can be represented as JSON is valid.  
     * Similar to [Document Expiration](../../../server/extensions/expiration), 
	   the metadata can be used to set the Compare Exchange item expiration.  
       Set `@expires` field to schedule expiration.  
       e.g. `{ "@expires": "2021-02-26T13:09:37.4040256Z" }`
  * **Raft index** - A version number that is modified on each change.  
    Any change to the value or metadata changes the Raft index.  

* Creating and modifying a compare-exchange item is an atomic, thread-safe [compare-and-swap](https://en.wikipedia.org/wiki/Compare-and-swap) interlocked 
  compare-exchange operation.

{PANEL/}

{PANEL: Creating and Managing Compare-Exchange Items}
  
Compare exchange items are created and managed with any of the following approaches:

* **Document Store Operations**  
  You can manage a compare-exchange item as an [Operation on the document store](../../../client-api/operations/compare-exchange/put-compare-exchange-value).  
  This can be done within or outside of a session (cluster-wide or single-node session).
   * When inside a session:  
     If the session fails, the compare-exchange operation can still succeed
     because store Operations do not rely on the success of the session.  
     You will need to delete the compare-exchange item explicitly upon session failure if you don't want the compare-exchange item to persist.

* **Cluster-Wide Sessions**  
  You can manage a compare-exchange item from inside a [Cluster-Wide session](../../../client-api/session/cluster-transaction/compare-exchange).  
  If the session fails, the compare-exchange item creation also fails.  
  None of the nodes in the group will have the new compare-exchange item.


* **Atomic Guards**  
  When creating documents using a cluster-wide session RavenDB automatically creates [Atomic Guards](../../../client-api/session/cluster-transaction/atomic-guards),  
  which are compare-exchange items that guarantee ACID transactions.  
  See [Cluster-wide vs. Single-node](../../../client-api/session/cluster-transaction/overview#cluster-wide-transaction-vs.-single-node-transaction) for a session comparision overview.

* **Studio**  
  Compare-exchange items can be created from the [Studio](../../../studio/database/documents/compare-exchange-view#the-compare-exchange-view) as well.

{PANEL/}

{PANEL: Why Compare-Exchange Items are Not Replicated to External Databases }

* Each cluster defines its policies and configurations, and should ideally have sole responsibility for managing its own documents. 
  Read [Consistency in a Globally Distributed System](https://ayende.com/blog/196769-B/data-ownership-in-a-distributed-system) 
  to learn more about why global database modeling is more efficient this way.
   
* When creating a compare-exchange item a Raft consensus is required from the nodes in the database group.
  Externally replicating such data is problematic as the target database may reside within a cluster that is in an
  unstable state where Raft decisions cannot be made. In such a state, the compare-exchange item will not be persisted in the target database.

* Conflicts between documents that occur between two databases are solved with the help of the documents
  Change-Vector. Compare-exchange conflicts cannot be handled properly as they do not have a similar
  mechanism to resolve conflicts.

* To ensure unique values between two databases without using compare-exchange items see [Example III](../../../client-api/operations/compare-exchange/overview#example-iii---ensuring-unique-values-without-using-compare-exchange).

{PANEL/}

{PANEL: Example I - Email Address Reservation}  

The following example shows how to use compare-exchange to create documents with unique values.  
The scope is within the database group on a single cluster. 

Compare-exchange items are not externally replicated to other databases.  
To establish uniqueness without using compare-exchange see [Example III](../../../client-api/operations/compare-exchange/overview#example-iii---ensuring-unique-values-without-using-compare-exchange).

{CODE email@Server\CompareExchange.cs /}  

**Implications**:

* The `User` object is saved as a document, hence it can be indexed, queried, etc.  

* This compare-exchange item was [created as an operation](../../../client-api/operations/compare-exchange/put-compare-exchange-value)
  rather than with a [cluster-wide session](../../../client-api/session/cluster-transaction/overview).  
  Thus, if `session.SaveChanges` fails, then the email reservation is _not_ rolled back automatically.  
  It is your responsibility to do so.  

* The compare-exchange value that was saved can be accessed in a query using `CmpXchg`:  
    {CODE-TABS}
    {CODE-TAB:csharp:Query-LINQ query_cmpxchg@Server\CompareExchange.cs /}  
    {CODE-TAB:csharp:Document-Query document_query_cmpxchg@Server\CompareExchange.cs /}  
    {CODE-TAB-BLOCK:sql:RQL}
    from Users as s where id() == cmpxchg("emails/ayende@ayende.com")  
    {CODE-TAB-BLOCK/}
    {CODE-TABS/}
    {PANEL/}

{PANEL: Example II - Reserve a Shared Resource}  

In the following example, we use compare-exchange to reserve a shared resource.  
The scope is within the database group on a single cluster.

The code also checks for clients who never release resources (i.e. due to failure) by using timeout.  

{CODE shared_resource@Server\CompareExchange.cs /}

{PANEL/}

{PANEL: Example III - Ensuring Unique Values without Using Compare Exchange}  

Unique values can also be ensured without using compare-exchange.

The below example shows how to achieve that by using **reference documents**.  
The reference documents' IDs will contain the unique values instead of the compare-exchange items.

Using reference documents is especially useful when [External Replication](../../../server/ongoing-tasks/external-replication) 
is defined between two databases that need to be synced with unique values.  
The reference documents will replicate to the destination database, 
as opposed to compare-exchange items, which are not externally replicated.

{NOTE: }
Sessions which process fields that must be unique should be set to [TransactionMode.ClusterWide](../../../client-api/session/cluster-transaction/overview).  
{NOTE/}

{CODE create_uniqueness_control_documents@Server\CompareExchange.cs /}
{PANEL/}

## Related Articles

### Client API

- [Get a Compare-Exchange Value](../../../client-api/operations/compare-exchange/get-compare-exchange-value)
- [Get Compare-Exchange Values](../../../client-api/operations/compare-exchange/get-compare-exchange-values)
- [Put a Compare-Exchange Value](../../../client-api/operations/compare-exchange/delete-compare-exchange-value)
- [Delete a Compare-Exchange Value](../../../client-api/operations/compare-exchange/delete-compare-exchange-value)
- [Atomic Guards](../../../client-api/session/cluster-transaction/atomic-guards)
- [Resolving Document Conflicts](../../../client-api/cluster/document-conflicts-in-client-side)


### Studio

- [Compare Exchange View](../../../studio/database/documents/compare-exchange-view)  

### Server

- [Conflict Resolution](../../../server/clustering/replication/replication-conflicts)
- [Cluster-Wide Transactions](../../../server/clustering/cluster-transactions)

---

## Code Walkthrough

- [Create CmpXchg Item](https://demo.ravendb.net/demos/csharp/compare-exchange/create-compare-exchange)  
- [Index CmpXchg Values](https://demo.ravendb.net/demos/csharp/compare-exchange/index-compare-exchange)  

---

## Ayende @ Rahien Blog

- [Consistency in a Globally Distributed System](https://ayende.com/blog/196769-B/data-ownership-in-a-distributed-system)


