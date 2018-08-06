# How to use the API

## Endpoints

The GraphQL endpoint is mounted on `/graphql/:path` where path is the url-encoded relative path of the Realm.

The subscription endpoint is `ws://ROS-URL:ROS-PORT/graphql/:path`.

The GraphiQL \(visual exploratory tool\) endpoint is mounted on `/graphql/explore/:path` where path, again, is the path of the Realm.

## Query

To query, you start with a `query` node. The schema of the Realm file is automatically used to generate the GraphQL schema that exposes the following operations:

* Query for all objects of a certain type: all object types have a pluralized node, e.g. `users`, `accounts`, etc. It accepts the following optional arguments:
  * `query`: a verbatim [realm.js query](https://realm.io/docs/javascript/latest/#filtering) that will be used to filter the returned dataset.
  * `sortBy`: a property on the object to sort by.
  * `descending`: sorting direction \(default is `false`, i.e. ascending\).
  * `skip`: offset to start taking objects from.
  * `take`: maximum number of items to return.
* Query for object by primary key: object types that have a primary key defined will have a singularized node, e.g. `user`, `realmFile`, etc. It accepts a single argument that is the primary key of the object.

{% hint style="warning" %}
For models which already have a plural name definition in your schema, you will need to add another "s" to your query.  \(i.e. "cars" becomes "carss", "data" becomes "datas", etc\)
{% endhint %}

For more information on how to use `query` see the [GraphQL documentation](http://graphql.org/learn/queries/).

## Mutation

To mutate an object, start with a `mutation` node. The schema of the Realm file is automatically used to generate the GraphQL schema that exposes the following operations:

* Add object: all object types have an `addObjectType` node, e.g. `addUser`, `addAccount`, etc. It accepts a single argument that is the object to add. Related objects will be added as well, e.g. specifying `accounts` in the `addUser` input will add the account objects.
* Add or update object: objects with primary key defined have an `updateObjectType` node, e.g. `updateUser`, `updateRealmFile`. It accepts a single argument that is the object to update. Partial updates are also allowed as long as the primary key value is specified.
* Delete objects:
  * Objects with primary key defined have a `deleteObjectType` node, e.g. `deleteUser`, `deleteRealmFile`. It accepts a single argument that is the primary key of the object to delete. Returns `true` if object was deleted, `false` otherwise.
  * All object types have a `deleteObjectTypes` node, e.g. `deleteUsers`, `deleteAccounts`, etc. It accepts a single optional argument - `query` that will be used to filter objects to be deleted. If not supplied, all objects of this type will be deleted.

For more information on how to use `mutation` see the [GraphQL documentation](http://graphql.org/learn/queries/#mutations).

## Subscription

To subscribe for change notifications, start with a `subscription` node. The schema of the Realm file is automatically used to generate the GraphQL schema that exposes the following operations:

* Subscribing for queries: all object types have a pluralized node, e.g. `users`, `accounts`, etc. Every time an item is added, deleted, or modified in the dataset, the updated state will be pushed via the subscription socket. The node accepts the following optional arguments:
  * `query`: a verbatim [realm.js query](https://realm.io/docs/javascript/latest/#filtering) that will be used to filter the returned dataset.
  * `sortBy`: a property on the object to sort by.
  * `descending`: sorting direction \(default is `false`, i.e. ascending\).
  * `skip`: offset to start taking objects from.
  * `take`: maximum number of items to return.



