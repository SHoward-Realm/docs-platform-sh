# Viewing your server logs

## Where can I find the server logs? 

While debugging your application, you'll fine the server logs to be very useful.  In 3.x versions, logging is enabled by default and is output to a `log.txt`file within the working directory of your ROS project.   If this file is not created, you can easily enable a logger by editing your server's index file like so: 

{% code-tabs %}
{% code-tabs-item title="index.ts" %}
```typescript
import { BasicServer, FileConsoleLogger } from 'realm-object-server'
import * as path from 'path'

const server = new BasicServer()

server.start({
    .
    .
    logger: new FileConsoleLogger(path.join(__dirname, '../log.txt'), 'all', {
                file: {
                    timestamp: true,
                    level: 'debug'
                },
                console: {
                    level: 'info'
                }
            }),
    .
    .
})
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## How do I increase the verbosity of my server logs? 

You can easily increase the verbosity of your server logs by editing the `level`property.  When troubleshooting, we typically recommend the `debug` level.  

{% hint style="warning" %}
In rare situations, it may help to increase your logging as high as `trace`but the sheer volume of output can quickly consume disk space.  
{% endhint %}

Levels include: 

* `fatal`
* `error`
* `warn`
* `info`
* `detail`
* `debug`
* `trace`



Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

