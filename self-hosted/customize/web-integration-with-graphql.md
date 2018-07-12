# Web Integration with GraphQL

Realm Object Server offers a web API through [GraphQL](http://graphql.org/). This enables retrieving Realm data in a web browser or in other backend language environments unsupported with Realm SDKs currently. The GraphQL API is provided through an additional service that can be run within the Realm Object Server alongside the default services \(sync, authentication, etc\) or it can be run within another Realm Object Server standalone.

The GraphQL service supports [query](http://graphql.org/learn/queries/) and [mutation](http://graphql.org/learn/queries/#mutations) via standard HTTP requests and realtime [subscription](https://github.com/facebook/graphql/blob/master/rfcs/Subscriptions.md) events via a websocket connection.

## Setup GraphQL Service {#setup-graphql-service}

To setup the GraphQL service you will need to either integrate it into an existing Realm Object Server project, or create another one specifically for the GraphQL service by running `ros init`:

```bash
ros init ros-graphql
```

Then, install the GraphQL service:

```bash
// Move into the ROS project
cd ros-graphql

// Install and save to package.json
npm install realm-graphql-service --save
```

Now we need to enable the service - open the `/src/index.ts` file and edit it:

```typescript
import { BasicServer } from 'realm-object-server'
import * as path from 'path'
import { GraphQLService } from 'realm-graphql-service'

const server = new BasicServer()

// Add the GraphQL service to ROS
server.addService(new GraphQLService({
    // Turn this off in production!
    disableAuthentication: true
}))

server.start({
      // Server configs...
    })
    .then(() => {
        console.log(`Realm Object Server was started on ${server.address}`)
    })
    .catch(err => {
        console.error(`Error starting Realm Object Server: ${err.message}`)
    })
```

Start the server with your GraphQL service:

```bash
npm start
```

You can verify that the service is running by opening up:[`http://localhost:9080/graphql/explore/%2F__admin`](http://localhost:9080/graphql/explore/%2F__admin) which displays [GraphiQL](https://github.com/graphql/graphiql), the in-browser IDE for exploring GraphQL. This url connects to your admin realm. Connect to a different realm like: `http://localhost:9080/graphql/explore/<realm_path>`

{% hint style="info" %}
For a detailed overview of the customization options, refer to the [GraphQL Service API Reference](https://realm.io/docs/realm-object-server/latest/api/graphql-service/).
{% endhint %}

## [How to use the API](../../graphql-web-access/how-to-use-the-api.md)

## [Setting up a basic GraphQL Client](../../graphql-web-access/using-the-graphql-client.md)

## Authentication {#authentication}

By default, all endpoints and actions are authenticated. If you wish to disable authentication while developing, you can pass `disableAuthentication: true` to the service constructor.If you're using the Realm GraphQL package for Apollo, authentication and token refreshes are handled automatically.

### Obtaining an Access Token

Authentication is done with an Access Token, obtained by executing POST request against the `/auth` endpoint.

First, you'll need to login the user with their provider. For example, when authenticating with username/password, pass the following payload:

```typescript
{
  "app_id":"",
  "provider":"password",
  "data":"MY-USERNAME",
  "user_info": {
    "register":false,
    "password":"MY-PASSWORD"
  }
}
```

The response will look something like:

```typescript
{
  "refresh_token": {
    "token":"VERY-LONG-TOKEN-HERE"
  }
}
```

We'll need the refresh token to obtain the access token by posting again to `/auth`:

```typescript
{
  "app_id":"",
  "provider":"realm", // Note provider is 'realm'
  "data":"REFRESH_TOKEN.TOKEN", // Token from previous response
  "path":"REALM-PATH" // Path of the realm you want to access, e.g. '/user-id/tickets
}
```

The response will now contain:

```typescript
{
  "access_token": {
    "token":"VERY-LONG-TOKEN-HERE"
  },
  "token_data": {
    "expires": 1512402618 // unix timestamp
  }
}
```

We'll need this access token to perform all graphql actions. This token must be refreshed before it expires using the refresh token obtained earlier to avoid getting `Unauthorized` responses.

### Query and Mutations

Since the query and mutation actions are regular GET/POST requests, the authentication happens with a standard `Authorization` header:

```typescript
Authorization: ACCESS_TOKEN.TOKEN
```

### Subscription

Subscriptions use websocket, which requires that authentication happens after the connection is established. Before sending any graphql-related messages, you'll need to send an object message, containing a `token` field set to the access token:

```typescript
{
  token: ACCESS_TOKEN.TOKEN
}
```

{% hint style="info" %}
**IMPORTANT NOTE ON TOKEN VALIDATION**: The access token for subscriptions is validated only when the socket connection is established and not when emitting notifications by the server. This means that it's the client's responsibility to terminate the connection if the user logs out or loses access to the Realm.
{% endhint %}

## API Reference {#api-reference}

The full GraphQL client API reference docs are located [here](https://realm.io/docs/realm-object-server/latest/api/graphql/).

The GraphQL Service API reference docs are [here](https://realm.io/docs/realm-object-server/latest/api/graphql-service/).

Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

