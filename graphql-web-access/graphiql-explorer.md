# GraphiQL Explorer

The GraphiQL explorer is a useful tool to explore the operations and their responses while developing. To run it, open your browser and navigate to[`http://localhost:9080/graphql/explore/%2F__admin`](http://localhost:9080/graphql/explore/%2F__admin) or [`https://your-cloud-url/graphql/explore/%2F__admin`](http://localhost:9080/graphql/explore/%2F__admin)- this will open the GraphiQL explorer for the `/__admin` Realm. If you'd like to browse another Realm, just replace the `%2F__admin` segment with the url-encoded relative path of the Realm.The GraphiQL explorer endpoints are subject to the same authentication rules as the regular GraphQL endpoints, so unless you've disabled authentication, you'll need to provide an authorization header. If you're using Chrome, you can use an extension, such as [ModHeader](https://chrome.google.com/webstore/detail/modheader/idgpnmonknjnojddfkpgkljpfnnfcklj) to inject the header into your requests.

### Querying {#querying}

To query, you start with a `query` node. All possible query nodes should be suggested by the autocompletion engine. You must explicitly specify all properties/relationships of the returned objects.

### Mutating {#mutating}

To mutate an object, start with a `mutation` node. All possible mutation methods should be suggested by the autocompletion engine. The returned values are objects themselves, so again, you should explicitly specify the properties you're interested in.

### Subscribing {#subscribing}

You can subscribe to a query by starting a `subscription` node. Whenever the Realm changes, an update will be pushed, containing the matching dataset. Additionally, immediately upon subscribing, the initial dataset will be sent.

### Inspecting the schema {#inspecting-the-schema}

You can see the schema of the Realm by querying the `__schema` node \(it will also include some built-in GraphQL types\):

```javascript
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

