# Configuration: Server Options

---

{NOTE: }

* Various configuration options for the server's behavior.  

* In this page:  
    * [Server.MaxTimeForTaskToWaitForDatabaseToLoadInSec](../../server/configuration/server-configuration#server.maxtimefortasktowaitfordatabasetoloadinsec)  
    * [Server.ProcessAffinityMask](../../server/configuration/server-configuration#server.processaffinitymask)  
    * [Server.IndexingAffinityMask](../../server/configuration/server-configuration#server.indexingaffinitymask)  
    * [Server.NumberOfUnusedCoresByIndexes](../../server/configuration/server-configuration#server.numberofunusedcoresbyindexes)  
    * [Server.CpuCredits.ExhaustionBackupDelayInMin](../../server/configuration/server-configuration#server.cpucredits.exhaustionbackupdelayinmin)  
    * [Server.Tcp.Compression.Disable](../../server/configuration/server-configuration#server.tcp.compression.disable)  

{NOTE/}

---

{PANEL:Server.MaxTimeForTaskToWaitForDatabaseToLoadInSec}

This setting is indicating how many seconds a task (e.g. request) will wait for the database to load (when it is unloaded - e.g. after server restart).

- **Type**: `int`
- **Default**: `30`
- **Scope**: Server-wide only

{PANEL/}

{PANEL:Server.ProcessAffinityMask}

EXPERT: The process affinity mask.

- **Type**: `long`
- **Default**: `null`
- **Scope**: Server-wide only

{PANEL/}

{PANEL:Server.IndexingAffinityMask}

EXPERT: The affinity mask to be used for indexing. Overrides the Server.NumberOfUnusedCoresByIndexes value. Should only be used if you also set `Server.ProcessAffinityMask`.

- **Type**: `long`
- **Default**: `null`
- **Scope**: Server-wide only

{PANEL/}

{PANEL:Server.NumberOfUnusedCoresByIndexes}

EXPERT: The numbers of cores that will be NOT running indexing. Defaults to 1 core that is kept for all other tasks and will not be used for indexing.

- **Type**: `int`
- **Default**: `1`
- **Scope**: Server-wide only

{PANEL/}

{PANEL:Server.CpuCredits.ExhaustionBackupDelayInMin}

**EXPERT:** When CPU credits are exhausted, backup tasks are canceled. This value 
determines how many minutes the server will wait before retrying the backup task.  

- **Type**: `TimeSetting`  
- **TimeUnit**: `TimeUnit.Minutes`  
- **Default**: `10`  
- **Scope**: Server-wide only  

{INFO: }
If you have an enterprise license, you can access information about CPU credits 
using [SNMP](../../server/administration/SNMP/snmp).  
{INFO/}

{PANEL/}


{PANEL:Server.Tcp.Compression.Disable}

Disable TCP Compression

- **Type**: `bool`
- **Default**: `false`
- **Scope**: Server-wide only

{PANEL/}
