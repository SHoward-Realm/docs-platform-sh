# Monitoring

Realm Object Server workers support sending metrics to **statsd**, which is assumed to be listening at `localhost:8125`. You can then forward these metrics to graphite or similar systems for monitoring.

All metrics keys start with a prefix of `realm.<hostname>`:

```text
realm.example.com.connection.online
realm.example.com.connection.failed
realm.example.com.realms.open
realm.example.com.protocol.violated
realm.example.com.protocol.bytes.received
realm.example.com.authentication.failed
```

**Metrics**

| Name | Type | Description |
| --- | --- | --- |
| `<prefix>.client.unsyncable` | counter | Triggered every time a client fails to initiate synchronization of a realm because of messed up history. Such clients need their realm file deleted and then recovered from the server. This might happen if the server crashes and is recovered from a backup. |
|  |  |  |
| `<prefix>.session.started` | counter | Triggered every time a session is started. A session is considered started even before the authentication. |
| `<prefix>.session.online` | gauge | The total number of sessions currently being served. |
| `<prefix>.session.failed` | counter | Triggered every time there is a session-level error. |
| `<prefix>.session.terminated` | counter | Triggered every time a session terminates. |
|  |  |  |
| `<prefix>.connection.started` | counter | Triggered every time a client opens a connection. |
| `<prefix>.connection.online` | gauge | The total number of connections currently open. Multiple sessions may be served through a connection. |
| `<prefix>.connection.failed` | counter | Low-level errors on connections. Triggered every time a failure happens during `accept()`, `read()` or `write()`. |
| `<prefix>.connection.terminated` | counter | Triggered every time a connection terminates. This includes the failed ones. |
|  |  |  |
| `<prefix>.realms.open` | gauge | The number of currently open realms. |
|  |  |  |
| `<prefix>.authentication.failed` | counter | Triggered on authentication failures, e.g. token invalid or expired. Should not normally happen. |
| `<prefix>.permission.denied` | counter | Triggered on permission failures, e.g. trying to access a realm with a token for another realm or trying to upload with a download-only token. Should not normally happen. |
| `<prefix>.protocol.<version>.used` | counter | Triggered every time a connection over protocol version `<version>` is established. Use this to track how many connections of each protocol version are initiated, and choose a better time to update the server and app. |
| `<prefix>.protocol.violated` | counter | Triggered every time the sync protocol is violated. May mean that the application is too old or badly written. |
|  |  |  |
| `<prefix>.protocol.bytes.received` | counter | Triggered every time an upload message is received by the server. |
| `<prefix>.protocol.bytes.sent` | counter | Triggered every time a download message is sent by the server. |
|  |  |  |
| `<prefix>.protocol.bytes.received` | gauge | The number of bytes received since the start. |
| `<prefix>.protocol.bytes.sent` | gauge | The number of bytes sent since the start. |

{% hint style="info" %}
In a production environment you would want to ship these metrics to a central repository so that you can programmatically trigger alerts or trigger deployment scripts. Realm provides a [walkthrough](https://realm.io/docs/tech-notes/rmp-walkthrough#adding-a-monitoring-system%20) on how to set-up Telegraf, InfluxDB, and Grafana for this but many others are available
{% endhint %}



Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

