# Load balancing client requests - Overview

 ---

{NOTE: }

* A database can have multiple instances, each one residing on a different cluster node.  
  Each instance is a complete replica of the database.  

* The [database-group-topology](../../../studio/database/settings/manage-database-group#database-group-topology---view) is the list of nodes that contain those database replicas.  
  The first node in this list is called the [preferred node](../../../client-api/configuration/load-balance/overview#the-preferred-node).

* The client is kept up-to-date with this topology list.  
  __The client decides which node from this list to access__ when making requests to the RavenDB cluster.  

* By default, the client will access the preferred node for all _Read & Write_ requests it makes.  
  This default behavior can be changed by configuring:
    * [ReadBalanceBehavior](../../../client-api/configuration/load-balance/read-balance-behavior) - load balancing `Read` requests only
    * [LoadBalanceBehavior](../../../client-api/configuration/load-balance/load-balance-behavior) - load balancing `Read & Write` requests  
  
---

* In this page:  
    * [Keeping the client topology up-to-date](../../../client-api/configuration/load-balance/overview#keeping-the-client-topology-up-to-date)
    * [Client logic for choosing a node](../../../client-api/configuration/load-balance/overview#client-logic-for-choosing-a-node)
    * [The preferred node](../../../client-api/configuration/load-balance/overview#the-preferred-node)
    * [Single-node session usage](../../../client-api/configuration/load-balance/overview#single-node-session-usage)
     
{NOTE/}

---

{PANEL: Keeping the client topology up-to-date}

* Upon Document Store initialization, the client receives the __initial topology list__,  
  after which the client is kept updated at all times for any changes made to it.

* If the topology list has changed on the server, (or any other client configuration),  
  the client will learn about it upon making its __next request__ to the server,  
  and will update its configuration accordingly.

* In addition, every 5 minutes, the client will fetch the current topology from the server  
  if no requests were made within that time frame.

* Any client-configuration settings that are set on the server side __override__ the settings made on the client-side.

* For more information see [Topology in the client](../../../client-api/cluster/how-client-integrates-with-replication-and-cluster#cluster-topology-in-the-client).  

{PANEL/}

{PANEL: Client logic for choosing a node}

The client uses the following logic (from top to bottom) to determine which node to send the request to:  

---

{INFO: }

* Use the __specified node__:   
  A client can explicitly specify the target node when executing a [server-maintenance operation](../../../client-api/operations/what-are-operations#server-maintenance-operations).  
  Learn more in [switch operation to a different node](../../../client-api/operations/how-to/switch-operations-to-different-node).  

---

* Else, if using-session-context is defined, use __LoadBalanceBehavior__:  
  Per session, the client will select a node based on the [session context](../../../client-api/configuration/load-balance/load-balance-behavior#loadbalancebehavior-options).  
  All `Read & Write` requests made on the session will be directed to that node.

---

* Else, if defined, use __ReadBalanceBehavior__:  
  `Read` requests: The client will select a node based on the [read balance options](../../../client-api/configuration/load-balance/read-balance-behavior#readbalancebehavior-options).  
  `Write` requests: All _Write_ requests will be directed to the preferred node. 
 
---

* Else, use the __preferred node__:  
  Use the [preferred node](../../../client-api/configuration/load-balance/overview#the-preferred-node) for both `Read & Write` requests.

{INFO/}

{PANEL/}

{PANEL: The preferred node}

* The preferred node is simply the __first__ node in the [topology nodes list](../../../studio/database/settings/manage-database-group#database-group-topology---view).  
* __By default__, when no load balancing strategy is defined,  
  the client will send all `Read & Write` requests to this node.  

---

* When the preferred node is in a failure state,  
  the cluster will update the topology, assigning another node to be the preferred one.  
* Once the preferred node is back up and has caught up with all data,  
  it will be placed __last__ in the topology list.  
* If all the nodes in the topology list are in a failure state then the first node in the list will be the 'preferred'.  
  The user would get an error, or recover if the error was transient.

---

* The preferred node can be explicitly set by:  
  * Reordering the topology list from the [Database Group view](../../../studio/database/settings/manage-database-group#database-group-topology---actions).  
  * Sending [ReorderDatabaseMembersOperation](../../../client-api/operations/server-wide/reorder-database-members) from the client code.  
* The cluster may assign a different preferred node when removing/adding new nodes to the database-group.  

{PANEL/}

{PANEL: Single-node session usage}

* When using a [single-node session](../../../client-api/session/cluster-transaction/overview#single-node),  
  a short delay in replicating changes to all nodes in the cluster is acceptable in most cases. 

* If `ReadBalanceBehavior` or `LoadBalanceBehavior` are defined,  
  then the next session you open may access a different node.  
  So if you need to ensure that the next request will be able to _immediately_ read what you just wrote,  
  then use [Write Assurance](../../../client-api/session/saving-changes#waiting-for-replication---write-assurance).  

{PANEL/}

## Related Articles

### Operations

- [What are operations](../../../client-api/operations/what-are-operations)

### Configuration

- [Load balance behavior](../../../client-api/configuration/load-balance/load-balance-behavior)
- [Read balance behavior](../../../client-api/configuration/load-balance/read-balance-behavior)
- [Conventions](../../../client-api/configuration/conventions)
- [Querying](../../../client-api/configuration/querying)
- [Serialization](../../../client-api/configuration/serialization)

### Client Configuration in Studio

- [Requests Configuration in Studio](../../../studio/server/client-configuration)
- [Requests Configuration per Database](../../../studio/database/settings/client-configuration-per-database)
- [Database-group-topology](../../../studio/database/settings/manage-database-group#database-group-topology---view)
