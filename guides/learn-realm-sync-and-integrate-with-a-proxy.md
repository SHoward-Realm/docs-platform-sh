# Learn Realm Sync and Integrate with a Proxy

## The Sync Protocol

Central to the Realm Platform is the Realm sync protocol which carries the operations of any changes to or from the Realm sync enabled client to the Realm Object Server. Opening a sync connection is a two step process.

First, the client will make a `login` API call as described [here](https://docs.realm.io/platform/using-synced-realms/user-authentication)

This is a POST to the Realm Object Server HTTP/HTTPS `/auth` endpoint containing the credentials of a the user or a valid user token. If the credentials or token are valid then the user is approved and the Realm Object Server issues its own token which contains the user’s Realm identity and their access within the Realm system. From here on out, the user is issued Realm internal access and refresh tokens and as long as the user is active the Realm system will continue to issue new refresh tokens until the initial authentication expires or the user goes inactive and must re-login again. 

After the initial authentication, the Realm client will have a valid Realm authorization token that it will use to initiate the Realm Sync connection which will enable bidirectional data flow build on a standards based Websocket. The first connection will be a GET to the Realm Object Server root endpoint which is a HTTP upgrade request in an attempt to establish a Websocket. If you took a look at the packet it would look like this:

```bash
GET /realm-sync/%2Fb920bd7068ef2b470ff4226e0e85716a%2Ffoo HTTP/1.1
Authorization: Realm-Access-Token version=1 token="..."
Connection: Upgrade
Host: myROS.internal.com
Sec-WebSocket-Key: FBeULDToSQx6zW+pXf8+Jw==
Sec-WebSocket-Protocol: io.realm.sync.22
Sec-WebSocket-Version: 13
Upgrade: websocket
```

## Adding a Custom Proxy

You will notice that the Authorization Header has the Realm Access Token embedded. This could pose a problem for proxies that may be expecting their own access token in this header. If this is the case you can use these APIs in Swift to change where the tokens are embedded.

To add your custom proxy token for your environment make a call like this to the SyncManager singleton:

```text
SyncManager.customRequestHeaders = [ "Authorization": myProxyToken ]
```

Now your Realm client traffic can flow unfettered through your already existing proxies, gateways, and firewalls. But the Realm system still needs to have a token to ensure that the connection is a valid Realm client and has permissions to a particular Realm. To do this we can add a new header onto the connection request like so:

```text
SyncManager.authorizationHeaderName = ‘X-Realm-Auth’
```

This will add an extra non-default header to the connection request so that the Realm system can continue to function in a secure manner. `X-Realm-Auth` is just a sample name for the header we chose but it can be whatever you choose to name it and be sure to configure your proxy not to strip this extra header or your Realm authentication will fail.

We also need to instruct the Realm Object Server on where to look for this Auth token that we configured the client to send in a new custom header. The `ServerConfig` in your `index.js` which runs the Realm Object Server gained a new property called `authorizationHeaderName` 

Set it to whatever you set the client header name to and pass it into the server `startConfig` like so:

```typescript
const startConfig = {
    services: [
        new ros.SyncProxyService(),
        authService,
        new ros.RealmDirectoryService(),
        new ros.PermissionsService(),
        new ros.LogService(),
    ],
    authorizationHeaderName: ‘X-Realm-Auth’,
    dataPath: path.join(__dirname, 'data'),

};

server.start(startConfig).then(() => {
    console.log(`Your server is started at ${server.address}`);
})
.catch(err => {
    console.error('There was an error starting your server');
    console.error(err);
});
```

The last thing you may need to integrate with your network is to format the GET request string.  In the above example it looks like this:

```text
/realm-sync/%2Fb920bd7068ef2b470ff4226e0e85716a%2Ffoo
```

You may have noticed that `/realm-sync/` has been prepended to the connection string which may break your proxy or API gateway that has whitelisted all url patterns after a certain point. After `/realm-sync/` comes the realm URL encoded path that tells ROS which Realm this client is trying to access, shown here: 

```text
%2Fb920bd7068ef2b470ff4226e0e85716a%2Ffoo
```

For instance your proxy may be set-up to allow connections to this URL:

```text
http://myCompany.org/my/url/prefix/<URL encoded path to Realm>
```

### Custom URL prefixes

To accommodate your environment which may have custom URL prefixes you can add urlPrefix to your SyncConfiguration like so:

```typescript
let syncConfiguration = SyncConfiguration(user: user!,
                                          realmURL: realmURL!,
                                          enableSSLValidation: true,
                                          isPartial: false,
                                          urlPrefix: "/my/url/prefix")
```





