# Server Side File Growth

## Background

Realm's conflict resolution algorithm utilizes transaction logs to allow clients who have been offline for extended periods of time to come back online and properly synchronize their data without conflicts. However, this benefit comes with an added overhead of larger file sizes on the server side. To fully support all offline clients, the transaction logs need to be kept around. You can read more about this [process in detail in our documentation on log compaction](../customize/log-compaction.md). To help manage this file growth, there are a number of solutions which are detailed below:

## History TTL Setting 

Over time as your app has had many, many sync transactions you may determine that it is no longer necessary to support very old clients \(i.e. a user who has been offline for months because they have uninstalled the application\). As a result, the server may discard some of these older transaction logs which will help minimize the overhead of the server's file storage size. This functionality is exposed by the `historyTtl` setting which can be configured from your server's `index.ts` file. This setting takes the maximum number of seconds which you want the transaction logs to live for. For example, if you only need to support clients which have connected and synced within the last week, you would set this setting to 604800 \(60 seconds \* 60 minutes \* 24 hours \* 7 days\). If a client has synced within the last week, they will be unaffected by the now discarded transaction logs. However, clients which have not connected and synced within the last week may experience a [client reset which will need to be handled by your application logic](../../using-synced-realms/troubleshoot/errors.md#client-reset). 

## Vacuum Utility

Realm stores its transactions within files on the server. Over time, these files can become fragmented as new transactions are too large to fit into previously allocated space in the file. When this happens, the file allocates more free disk space for the transaction and as a result a small amount of space may go unused in the original file. Over time, this unused space can add up and be wasteful as well as decrease performance. As a result, we offer a "vacuum" utility in ROS version 3.6.0 and newer. This utility removes the unused space and effectively compacts the transaction logs and state. 

### Using the utility:

1. The tool is located within your working ROS directory such as:`./node_modules/realm-sync-server/compiled/{darwin-x64|linux-x64}/bin/realm-vacuum`
2. Before running the tool, make sure to [backup your server files](enterprise-architecture/backup.md). 
3. Stop the Realm Object Server process 
4. You can start by performing a dry run with the utility.  This will give you an estimate of how much space might be saved by running the utility via the terminal: `realm-vacuum --dry-run <path-to-your-big-server-file>`
5. You can then run the utility itself like:

   `realm-vacuum --dry-run <path-to-your-big-server-file>`

You should see an output like the following: 

```text
File:   /home/ubuntu/rosdata/sync/user_data/path/myrealm.realm
Type:   Sync Server
Before: beforesize bytes ( beforesize GB )
After:  aftersize bytes ( aftersize GB
Change: -delta% (dry run; no modifications made)
```



