# Version Compatibilities

Realm Object Server 3.x is backwards compatible with all SDK's that supports ROS 3.0 or later. So when a new ROS version is available, always update the server first and then the client SDKs.

While ROS supports older SDK's than the ones listed below, the versions listed are required to support the new ROS 3.0 features like [Query-based Synchronization](../syncing-data.md#using-partial-synchronization) and [Fine-grained Permissions](../access-control/#fine-grained-permissions-1).

With ROS 3.11 a new capability to compress server side files was introduced which requires updated SDK version as listed below.

| ROS | Realm JS | Realm Swift + Objective-C | Realm Java | Realm .Net | Realm Data Adapter | Realm Studio |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 3.0 - 3.10 | &gt;= 2.3 | &gt;= 3.2 | &gt;= 5.0 | &gt;= 3.0 | &gt;= 1.1 | &gt;= 1.0 |
| 3.11 - | &gt;= 2.17 | &gt;= 3.10 | &gt;= 5.7.0 | TBD | TBD | &gt;= 3.0 |

{% hint style="info" %}
Node.js 6.0 or later is required for installation of the server.
{% endhint %}

Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

