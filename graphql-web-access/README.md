# GraphQL - Web Access

## Overview

Realm Platform offers a GraphQL API which enables retrieving Realm data in a web browser or in other backend language environments unsupported with Realm SDKs currently. The API supports retrieving data or applying changes via HTTP requests and realtime [subscription](https://github.com/facebook/graphql/blob/master/rfcs/Subscriptions.md) events via a Websockets connection.

### Why GraphQL?

When we were first planning to add web integration, our first thought was to generate a REST API based off the Realms and their schemas in the server. However, one of the limitations of standard REST APIs is that the server defines them. This becomes problematic when you have an object-graph data model, like Realm. For example lets say you have two models in your Realm: `Person` and `Dog`:

```swift
// Dog model
class Dog: Object {
    @objc dynamic var name = ""
    
    override static func primaryKey() -> String? {
        return "name"
    }
}

// Person model
class Person: Object {
    @objc dynamic var name = ""
    @objc dynamic var birthdate = Date(timeIntervalSince1970: 1)
    let dogs = List<Dog>()
}
```

Serializing a Realm object into JSON means you have to decide on whether you include the linked objects. In the example, if you had a REST API `/persons` which would return all `Person` objects from the Realm, do you include the related `Dog` objects within the JSON or do you just include the primary keys? This becomes more problematic the deeper your object-graph becomes.

GraphQL offers an elegant solution to this problem by defining a query language that a client can use with a single HTTP endpoint that allows it to define the API action rather than the server. This ends up mapping perfectly with Realms' data model. Following the example above here is how you could request `Person` objects from the GraphQL API:

```javascript
query {
    persons {
        name
        birthdate
        dogs {
            name        
        }
}
```

You can learn more about the syntax later, the key point is that the client is able to specify which fields it wants returned. As a result, we can specify that we want the `Dog` objects and we also want to include their `name` field values. With a complex object-graph this allows the client the freedom to specify just the right amount of data needed, reducing unnecessary roundtrips to look up related objects by IDs or sending wasteful data that is unneeded.

### Subscriptions

In addition to the benefits that GraphQL mates well with Realm's data model for standard read or write operations, there is another feature of GraphQL that ultimately completes the pairing: [Subscriptions](http://graphql.org/blog/subscriptions-in-graphql-and-relay/). This operation is designed to allow the client to subscribe to data change events. It isn't as common with GraphQL implementations because it requires a reactive architecture to power the data change events. With Realm Platform, though, reactivity is standard! Realm objects are _"live objects"_ that automatically update in response to changes across threads, processes, or with Realm Platform devices. Our APIs offer the ability to register callbacks for not only events that the data changed but also metadata on what changes occurred.

This meant that adding GraphQL subscriptions was seamless. Clients can subscribe for queries which establishes a Websockets connection with the client so that data change events can be pushed from the server. As a result, with Realm Platform you can develop a web application using the GraphQL API and provide the same realtime reactivity that our Realm SDKs provide for mobile!

