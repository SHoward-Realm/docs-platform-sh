# Using the GraphQL Client

Since the GraphQL API is just a regular HTTP API, any http client can be used to query it. To make it easier to get started, Realm provides a few helper and convenience API for the [Apollo Client](https://www.apollographql.com/client), a popular Javascript client that supports a variety of web frameworks as well as Node.js.

Here's a minimal getting started example:

```javascript
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
Looking for the client API reference docs? They're [here.](https://realm.io/docs/realm-object-server/latest/api/graphql/)
{% endhint %}

### Prerequisites {#prerequisites}

Add the [apollo-client-preset](https://www.npmjs.com/package/apollo-client-preset), [apollo-link-ws](https://www.npmjs.com/package/apollo-link-ws), and [subscriptions-transport-ws](https://www.npmjs.com/package/subscriptions-transport-ws) packages to your project:

```bash
npm install apollo-client-preset apollo-link-ws subscriptions-transport-ws --save
```

Then, add the Realm GraphQL client package:

```bash
npm install realm-graphql-client --save
```

### Authenticating the user  {#authenticating-the-user}

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
var credentials = Credentials.UsernamePassword('SOME-USERNAME', 'SOME-PASSWORD' createUser: false);
var authURL = new Uri('http://my-ros-instance:9080');
var user = await User.LoginAsync(credentials, authURL);
```
{% endtab %}
{% endtabs %}

Other credential providers are supported, such as JWT, Facebook, Google, etc. They are all exposed as factories on the [Credentials](file:///Users/matt/Documents/GitHub/realm.io/source/en/docs/realm-object-server/2.0/api/graphql/classes/credentials.html) class. If you passed `disableAuthentication: true` when creating the GraphQL service, you can also authenticate with an [anonymous](https://docs.realm.io/platform/v/3.x/self-hosted/develop/integration/api/graphql/classes/credentials.html#anonymous) user:

```javascript
const credentials = Credentials.anonymous();
const user = await User.authenticate(credentials, 'http://my-ros-instance:9080');
```

After you have your user, you can create a [GraphQLConfig](file:///Users/matt/Documents/GitHub/realm.io/source/en/docs/realm-object-server/2.0/api/graphql/classes/graphqlconfig.html) that will handle token refreshes and authentication:

```javascript
const config = await GraphQLConfig.create({ 
  user: user,
  realmPath: `/~/test`
});
```

Note that each config is created per Realm path, so if you need to query multiple Realms, you'll need to obtain a config instance for each of them.

### Setting up the Client {#setting-up-the-client}

Once you have a config, you can use that to create an Apollo client instance and configure it. The config exposes 4 properties:

* `httpEndpoint`: This is the endpoint you'll use to execute queries and mutations against. It can be used to configure Apollo's [httpLink](https://www.apollographql.com/docs/link/links/http.html).
* `authLink`: This is a link that provides an Authorization header for the user/path combination. It should be composed together with your `httpLink`.
* `webSocketEndpoint`: This is the endpoint you'll use to execute subscriptions against. It can be used to configure Apollo's [WebSocket Link](https://www.apollographql.com/docs/link/links/ws.html).
* `connectionParams`: This is a function that will provide an authorization object each time a websocket connection is established. You should pass that directly \(without invoking it\) to the WebSocketLink's constructor's options.

```javascript
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

### Queries {#queries}

Querying data is as simple as invoking `client.query()`:

```javascript
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

For a complete list of supported query operations, refer to the [GraphQL Server docs](https://docs.realm.io/platform/v/3.x/self-hosted/develop/integration/web-integration#query).

For a detailed documentation on the Apollo Client query capabilities, refer to the [Apollo docs](https://www.apollographql.com/docs/angular/basics/queries.html).

### Mutations {#mutations}

Mutating data happens when you invoke the `client.mutate()`method:

```javascript
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

For a complete list of supported mutation operations, refer to the [GraphQL Server docs](https://docs.realm.io/platform/v/3.x/self-hosted/develop/integration/web-integration#mutation).

For a detailed documentation on the Apollo Client mutation capabilities, refer to the [Apollo docs](https://www.apollographql.com/docs/angular/basics/mutations.html).

### Subscriptions {#subscriptions}

Subscribing for changes happens when you invoke the `client.subscribe()` method. You get an `Observable` sequence you can then add an observer to:

```javascript
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

## Errors: 

### "Cannot access realm that has been closed."

Realm's GraphQL Service works by querying cached Realm files from the server. While these cached files should always be available, there are a few rare conditions where they may get closed. In this event, you'll want to reach out to our [support team](https://www.support.realm.io).  When you do this please provide the following: instance information \(i.e. cloud URL or self-hosting version information for all packages\) and a reproduction case.

