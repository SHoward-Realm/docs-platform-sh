# Log Compaction

## Background

Delivering realtime performance with Realm's data sync begins by taking advantage of a capability within Realm Database. Realm's multi-threading/process capabilities require the database to maintain a transaction log to coordinate operations across these boundaries. The log includes all the individual changes made to the database and when using a synchronized Realm, these operations are transmitted in the sync protocol.

This approach to data sync offers several benefits:

* Less bandwidth and faster transmission by only sending the specific changes \(as compared to a state-based system that might require resending the full modified object\)
* Semantically correct, for example an operation such as a move in Realm's ordered `List` property is not transmitted as an insert and delete

The complete operation log for a synchronized Realm is maintained by the Realm Object Server. Whenever a client performs a change to a synchronized Realm, the database updates and produces a local log entry which is transmitted to the server and then incorporated into the complete log. The server sends the client an acknowledgement of this, which allows the client to prune its local copy of the log entry. This means that client devices don't have to maintain the entire history, keeping the storage space used by the application to a minimum.

So while the client can continually prune entries, the server must maintain the complete history to be able to coordinate sync across devices at different states. This is most prominent when a new device begins syncing for the first time and needs to incorporate the entire log to be in sync with up-to-date devices. To prevent the need to send the entire history, Realm Object Server, offers log compaction.

## How Does It Work?

The log compaction algorithm runs alongside of the server process that merges and synchronizes changes. This allows the server to identify sequential operations that can be compacted but still result in the same output. For example, if you created an object, deleted, then re-created it again, these three operations can be compacted into a single create operation.

By default, the server has log compaction enabled but it can be disabled via server configuration:

```javascript
import { BasicServer } from 'realm-object-server'
import * as path from 'path'

const server = new BasicServer()

server.start({
    // Default: true
    enableDownloadLogCompaction: false
})
```

To control how the log compaction algorithm operates there is another server configuration option:

```javascript
maxDownloadSize: 16000000 // 16MB
```

The `maxDownloadSize` value represents the size of changes that will be considered as input to compaction. Generally, Realm Object Server can't discover how much space is saved by compaction without actually running the compaction algorithm, which is a CPU-heavy task. It represents a "worst case" download size, which is when the compaction algorithm couldn't eliminate anything, but on the other hand more changes will be considered the larger it is, so the net result is that the smallest total number of bytes downloaded will decrease as the `maxDownloadSize` value increases.

Thus, `maxDownloadSize` is a compromise between download size and CPU usage. Typically, this value might need adjusted if you need to tune the initial download time of a synchronized Realm when a new client device connects.

## Enterprise Edition

When running the Enterprise Edition, you do not need to adjust the log compaction for the "Core Services" ROS. Instead, the log compaction configuration _only_ applies to the ROS instances that have a sync service. For example:

```typescript
const startConfig = {
    services: [
        new ros.LogService(),
        new rose.ReplicatedSyncService({
            // ... other configs
            maxDownloadSize: 16000000,
            enableDownloadLogCompaction: true
        })
    ],
    // Other server configs
    // The log compaction settings don't need to be duplicated here
}

server.start(startConfig).then(() => {
    console.log(`Your server is started `, server.address)
})
.catch(err => {
    console.error(`There was an error starting your file`)
})
```



Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

