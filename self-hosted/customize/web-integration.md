# Web Integration

  
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

You can verify that the service is running by opening up:[`http://localhost:9080/graphql/explore/%2F__admin`](http://localhost:9080/graphql/explore/%2F__admin) which displays [GraphiQL](https://github.com/graphql/graphiql), the in-browser IDE for exploring GraphQL.  This url connects to your admin realm.  Connect to a different realm like: `http://localhost:9080/graphql/explore/<realm_path>`

{% hint style="info" %}
For a detailed overview of the customization options, refer to the [GraphQL Service API Reference](https://realm.io/docs/realm-object-server/latest/api/graphql-service/).
{% endhint %}

### Endpoints {#endpoints}

The GraphQL endpoint is mounted on `/graphql/:path` where path is the url-encoded relative path of the Realm.

The subscription endpoint is `ws://ROS-URL:ROS-PORT/graphql/:path`.

The GraphiQL \(visual exploratory tool\) endpoint is mounted on `/graphql/explore/:path` where path, again, is the path of the Realm.

### Query {#query}

To query, you start with a `query` node. The schema of the Realm file is automatically used to generate the GraphQL schema that exposes the following operations:

* Query for all objects of a certain type: all object types have a pluralized node, e.g. `users`, `accounts`, etc. It accepts the following optional arguments:
  * `query`: a verbatim [realm.js query](https://realm.io/docs/javascript/latest/#filtering) that will be used to filter the returned dataset.
  * `sortBy`: a property on the object to sort by.
  * `descending`: sorting direction \(default is `false`, i.e. ascending\).
  * `skip`: offset to start taking objects from.
  * `take`: maximum number of items to return.
* Query for object by primary key: object types that have a primary key defined will have a singularized node, e.g. `user`, `realmFile`, etc. It accepts a single argument that is the primary key of the object.

For more information on how to use `query` see the [GraphQL documentation](http://graphql.org/learn/queries/).

### Mutation {#mutation}

To mutate an object, start with a `mutation` node. The schema of the Realm file is automatically used to generate the GraphQL schema that exposes the following operations:

* Add object: all object types have an `addObjectType` node, e.g. `addUser`, `addAccount`, etc. It accepts a single argument that is the object to add. Related objects will be added as well, e.g. specifying `accounts` in the `addUser` input will add the account objects.
* Add or update object: objects with primary key defined have an `updateObjectType` node, e.g. `updateUser`, `updateRealmFile`. It accepts a single argument that is the object to update. Partial updates are also allowed as long as the primary key value is specified.
* Delete objects:
  * Objects with primary key defined have a `deleteObjectType` node, e.g. `deleteUser`, `deleteRealmFile`. It accepts a single argument that is the primary key of the object to delete. Returns `true` if object was deleted, `false` otherwise.
  * All object types have a `deleteObjectTypes` node, e.g. `deleteUsers`, `deleteAccounts`, etc. It accepts a single optional argument - `query` that will be used to filter objects to be deleted. If not supplied, all objects of this type will be deleted.

For more information on how to use `mutation` see the [GraphQL documentation](http://graphql.org/learn/queries/#mutations).

### Subscription {#subscription}

To subscribe for change notifications, start with a `subscription` node. The schema of the Realm file is automatically used to generate the GraphQL schema that exposes the following operations:

* Subscribing for queries: all object types have a pluralized node, e.g. `users`, `accounts`, etc. Every time an item is added, deleted, or modified in the dataset, the updated state will be pushed via the subscription socket. The node accepts the following optional arguments:
  * `query`: a verbatim [realm.js query](https://realm.io/docs/javascript/latest/#filtering) that will be used to filter the returned dataset.
  * `sortBy`: a property on the object to sort by.
  * `descending`: sorting direction \(default is `false`, i.e. ascending\).
  * `skip`: offset to start taking objects from.
  * `take`: maximum number of items to return.

## Consuming the GraphQL API {#consuming-the-graphql-api}

Since the GraphQL API is just a regular HTTP API, any http client can be used to query it. To make it easier to get started, Realm provides a few helper and convenience API for the [Apollo Client](https://www.apollographql.com/client), a popular javascript client that supports a variety of web frameworks as well as node.js.

Here's a minimal getting started example:

```typescript
const credentials = Credentials.usernamePassword('SOME-USERNAME', 'SOME-PASSWORD');
const user = await User.authenticate(credentials, 'http://my-ros-instance:9080');

const config = await GraphQLConfig.create({ 
  user: user,
  realmPath: '/~/test'
});

const httpLink = concat(
    config.authLink,
    // Note: if using node.js, you'll need to provide fetch as well.
    new HttpLink({ uri: config.httpEndpoint })
  );

// Note: if using node.js, you'll need to provide webSocketImpl as well.
const subscriptionLink = new WebSocketLink({
  uri: config.webSocketEndpoint,
  options: {
    connectionParams: config.connectionParams,
  }
});

// Send subscription operations to the subscriptionLink and
// all others - to the httpLink
const link = split(({ query }) => {
    const { kind, operation } = getMainDefinition(query);
    return kind === 'OperationDefinition' && operation === 'subscription';
  },
  subscriptionLink,
  httpLink
);

client = new ApolloClient({
  link: link,
  cache: new InMemoryCache()
});

// You can now query the GraphQL API
const response = await client.query({
  query: gql`
    query {
      people(query: "age > 18", sortBy: "name") {
        name
        age
      }
    }
  `
});

const people = response.data.people;
```

{% hint style="info" %}
Looking for the client API reference docs? They're here. 
{% endhint %}

### Prerequisites

Add the [apollo-client-preset](https://www.npmjs.com/package/apollo-client-preset), [apollo-link-ws](https://www.npmjs.com/package/apollo-link-ws), and [subscriptions-transport-ws](https://www.npmjs.com/package/subscriptions-transport-ws) packages to your project:

```bash
npm install apollo-client-preset apollo-link-ws subscriptions-transport-ws --save
```

Then, add the Realm GraphQL client package:

```bash
npm install realm-graphql-client --save
```

### Authenticating the user 

To start consuming the GraphQL API, you'll need to login a [User](file:///Users/matt/Documents/GitHub/realm.io/source/en/docs/realm-object-server/2.0/api/graphql/classes/user.html):

{% tabs %}
{% tab title="Node.js" %}
```javascript
const credentials = Credentials.usernamePassword('SOME-USERNAME', 'SOME-PASSWORD');
const user = await User.authenticate(credentials, 'http://my-ros-instance:9080');
```
{% endtab %}

{% tab title="C\#" %}
```csharp
var credentials = Credentials.UsernamePassword('SOME-USERNAME', 'SOME-PASSWORD' createUser: false);
var authURL = new Uri('http://my-ros-instance:9080');
var user = await User.LoginAsync(credentials, authURL);
```
{% endtab %}
{% endtabs %}

Other credential providers are supported, such as JWT, Facebook, Google, etc. They are all exposed as factories on the [Credentials](file:///Users/matt/Documents/GitHub/realm.io/source/en/docs/realm-object-server/2.0/api/graphql/classes/credentials.html) class. If you passed `disableAuthentication: true` when creating the GraphQL service, you can also authenticate with an [anonymous](api/graphql/classes/credentials.html#anonymous) user:

```typescript
const credentials = Credentials.anonymous();
const user = await User.authenticate(credentials, 'http://my-ros-instance:9080');
```

After you have your user, you can create a [GraphQLConfig](file:///Users/matt/Documents/GitHub/realm.io/source/en/docs/realm-object-server/2.0/api/graphql/classes/graphqlconfig.html) that will handle token refreshes and authentication:

```typescript
const config = await GraphQLConfig.create({ 
  user: user,
  realmPath: `/~/test`
});
```

Note that each config is created per Realm path, so if you need to query multiple Realms, you'll need to obtain a config instance for each of them.

### Setting up the Client

Once you have a config, you can use that to create an Apollo client instance and configure it. The config exposes 4 properties:

* `httpEndpoint`: This is the endpoint you'll use to execute queries and mutations against. It can be used to configure Apollo's [httpLink](https://www.apollographql.com/docs/link/links/http.html).
* `authLink`: This is a link that provides an Authorization header for the user/path combination. It should be composed together with your `httpLink`.
* `webSocketEndpoint`: This is the endpoint you'll use to execute subscriptions against. It can be used to configure Apollo's [WebSocket Link](https://www.apollographql.com/docs/link/links/ws.html).
* `connectionParams`: This is a function that will provide an authorization object each time a websocket connection is established. You should pass that directly \(without invoking it\) to the WebSocketLink's constructor's options.

```typescript
const httpLink = concat(
    config.authLink,
    // Note: if using node.js, you'll need to provide fetch as well.
    new HttpLink({ uri: config.httpEndpoint })
  );

// Note: if using node.js, you'll need to provide webSocketImpl as well.
const subscriptionLink = new WebSocketLink({
  uri: config.webSocketEndpoint,
  options: {
    connectionParams: config.connectionParams,
  }
});
```

### Queries

Querying data is as simple as invoking `client.query()`:

```typescript
const response = await client.query({
  query: gql`
    query {
      companies {
        companyId
        name
        address
      }
    }
  `
});

const companies = response.data.companies;
```

For a complete list of supported query operations, refer to the [GraphQL Server docs](#query).

For a detailed documentation on the Apollo Client query capabilities, refer to the [Apollo docs](https://www.apollographql.com/docs/angular/basics/queries.html).

### Mutations

Mutating data happens when you invoke the `client.mutate()`method:

```typescript
const response = await client.mutate({
  mutation: gql`
    mutation {
      result: addCompany(input: {
        companyId: "some-unique-id"
        name: "My Amazing Company"
        address: "Mars"
      }) {
        companyId
        name
        address
      }
    }
  `
});

const addedCompany = response.data.result;
```

For a complete list of supported mutation operations, refer to the [GraphQL Server docs](#mutation).

For a detailed documentation on the Apollo Client mutation capabilities, refer to the [Apollo docs](https://www.apollographql.com/docs/angular/basics/mutations.html).

### Subscriptions

Subscribing for changes happens when you invoke the `client.subscribe()` method. You get an `Observable` sequence you can then add an observer to:

```typescript
const observable = await client.subscribe({
  query: gql`
    subscription {
      companies {
        companyId
        name
        address
      }
    }
  `
});

observable.subscribe({
  next(data) {
    const companies = data.data.companies;
    // Update you UI
  },
  error(value) {
    // Notify the user of the failure
  }
});
```

For a complete list of supported subscription operations, refer to the GraphQL Server docs.

For a detailed documentation on the Apollo Client subscription capabilities, refer to the [Apollo docs](https://www.apollographql.com/docs/angular/features/subscriptions.html).

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

## Exploring with GraphiQL {#exploring-with-graphiql}

The GraphiQL explorer is a useful tool to explore the operations and their responses while developing. To run it, open your browser and navigate to[`http://localhost:9080/graphql/explore/%2F__admin`](http://localhost:9080/graphql/explore/%2F__admin) - this will open the GraphiQL explorer for the `/__admin` Realm. If you'd like to browse another Realm, just replace the `%2F__admin` segment with the url-encoded relative path of the Realm.The GraphiQL explorer endpoints are subject to the same authentication rules as the regular GraphQL endpoints, so unless you've disabled authentication, you'll need to provide an authorization header. If you're using Chrome, you can use an extension, such as \[ModHeader\]\(https://chrome.google.com/webstore/detail/modheader/idgpnmonknjnojddfkpgkljpfnnfcklj\) to inject the header into your requests.

### Querying

To query, you start with a `query` node. All possible query nodes should be suggested by the autocompletion engine. You must explicitly specify all properties/relationships of the returned objects.

### Mutating

To mutate an object, start with a `mutation` node. All possible mutation methods should be suggested by the autocompletion engine. The returned values are objects themselves, so again, you should explicitly specify the properties you're interested in.

### Subscribing

You can subscribe to a query by starting a `subscription` node. Whenever the Realm changes, an update will be pushed, containing the matching dataset. Additionally, immediately upon subscribing, the initial dataset will be sent.

### Inspecting the schema

You can see the schema of the Realm by querying the `__schema` node \(it will also include some built-in GraphQL types\):

```typescript
{
  __schema {
    types {
      name
      fields {
        name
      }
    }
  }
}
```

## API Reference {#api-reference}

The full GraphQL client API reference docs are located [here](https://realm.io/docs/realm-object-server/latest/api/graphql/).

The GraphQL Service API reference docs are [here](https://realm.io/docs/realm-object-server/latest/api/graphql-service/).





Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

