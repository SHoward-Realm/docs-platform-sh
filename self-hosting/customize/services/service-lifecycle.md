# Service Lifecycle

When designing services, it's important to know the lifecycle of the service. Services have 3 events

1. an optional `async start` method that can be decorated by `@Start`
   * If you have isolated asynchronous setup code use this method.
   * You shouldn't be calling other services.
   * The server object is passed as the only argument, where you can grab the logger, discovery, or any other global configuration needed by the service.
   * Prefer not to block the execution of this method.
2. an optional `async serverStarted` method that can be decorated by `@ServerStarted`
   * At this stage, the server has successfully started all services that were added.
3. an optional `async stop` method that can be decorated by `@Stop`
   * This is called when the `shutdown` command is called on the server.
   * Use this method to quickly dispose or close any connections.

Note, all these decorators are completely optional.

```text
class SampleService {

    @Start
    async start(server: Server) {
        // called when the server is about to startup
    }

    @ServerStarted
    async serverStarted(server: Server) {
        // called when all services have started and the underlying httpServer is started
    }

    @Stop
    async stop(){
        // do your service cleanup here
        // called when the server is attempting to shutdown
    }
}
```





Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

