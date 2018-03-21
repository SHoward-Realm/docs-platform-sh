# Discovering Services

For an Enterprise Edition \(EE\) deployment we utilize Consul for service discovery. It offers a consistent data store that can be run highly available, which we use to then store the information about which services and their URLs are participating in the cluster.

In this manner, each service can then lookup through the service discovery \(which itself does a look up in the data store of Consul\) to find out if a service is available and what the URL is for the primary node of that service is.

Let's use an example from the ROS code to see how this works. In the [`AuthService`](https://github.com/realm/realm-object-server-private/blob/master/src/services/AuthService.ts) it needs to be able to lookup the location of a given Realm file, specifically what sync worker it is located on--called the `syncLabel`. This information is maintained by the `RealmDirectoryService`. As a result, the `AuthService` has to use the service discovery system to find the `RealmDirectoryService` then call the endpoint on this service it needs.

Each service can register a function that is called when the `Server` starts. It will be passed in the `server` object which contains the `discovery` object:

```typescript
@Start()
    async start(server: Server) {
        this.logger = server.logger.withContext({ service: "auth" });
        this.discovery = server.discovery;
```

Now that the service is holding a reference to the `discovery` object it can use it within its own logic. Looking at the method that generates an access token we see that the `AuthService` must call the `RealmDirectoryService`. The first thing is that it must lookup the `RealmDirectoryService` via `discovery`. The `RealmDirectoryService` uses the name `realms`:

```typescript
async accessToken(path: string, data: any, app_id?: string): Promise<RealmAccessTokenResponse> {

    ...

    let realmsHandle = await this.discovery.find('realms');
```

This call returns a `handle` which provides the location of the `RealmDirectoryService`, which it can then use to call out to it:

```typescript
const response = await this.findRealmByPathUrl(realmsHandle, (partialSyncPath || path), data)
const syncLabel = response.body['syncLabel'] as string
```

Looking at the implementation of `findRealmByPathUrl()` function we see the specifics:

```typescript
private async findRealmByPathUrl(handle: ServiceHandle, path: string, token: any): Promise<any> {
    const findRealmByPathUrl = new URI(`http://${handle.address}:${handle.port}/`)
    .segment('/realms')
    .segment('/files')
    .segment(encodeURIComponent(path))
    .toString()

    return await superagent.get(findRealmByPathUrl)
    .set({
        'X-Realm-Access-Token': token
    })
}
```

The `handle.address` and `handle.port` are used to build the REST request from the `AuthService` to the `RealmDirectoryService`. 

This basic flow means that any additional service can perform similar actions in an EE cluster environment. Each service has a name and service discovery via Consul keeps track of each one.





Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

