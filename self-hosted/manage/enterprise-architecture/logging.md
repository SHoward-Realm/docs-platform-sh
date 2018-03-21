# Logging

All components of the Realm Object Server output logs to a standard interface which can then be piped to the console or more commonly to a file on disk.  We like to recommend pm2 for more granular logging capabilities during development or testing.  

However, in a production system, it is a best practice to offload the logs to a central logging repository which has more refined tools for detecting anomalies and performing root cause analysis. There are a variety of tools on the market today, Realm provides a [walkthrough](https://realm.io/docs/tech-notes/rmp-walkthrough#adding-a-logging-system) on how to set-up one such free system with Elasticsearch, Filebeat, and Kibana. See here for details:

## Using PM2 for Logging for a single server

You can use pm2 to monitor the usage of ROS instances by:

`pm2 monit`

This will print a GUI that will display both logs, CPU usage, and Memory usage.

pm2 also includes robust log management. You can use it to display logs in realtime in your console, log to a file, and rotate logs. Please see the [pm2 Log Management documentation](http://pm2.keymetrics.io/docs/usage/log-management/) to set this up.

In addition, Realm Object Server includes built-in logging capabilities as well. You can supply a custom logger to the server via a start parameter. The server includes three logging classes for this:

`ConsoleLogger`

This class will just log directly to console. It is included in the default configuration of the server, so no changes are needed to use it.

`FileLogger`

This class will log directly to a file, but not the console. This cannot be used with pm2 logging. To use add it to your server start function:

```typescript
server.start({
  logger: new FileLogger('myfile.log', 'info')
})
```

`FileConsoleLogger`

Finally, this class will log to the console and a file. To use add it to your server start function:

```typescript
server.start({
  logger: new FileConsoleLogger('myfile.log', 'info')
})
```

Not what you were looking for? [Leave Feedback](mailto:docs-feedback@realm.io)

