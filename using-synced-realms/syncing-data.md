# Syncing Data

## Overview

Data can be synchronized between a local Realm and the server in two modes: _Partial \_and_ Full \_ synchronization.

* **Query-based synchronization** is a feature that allows a synchronized Realm to be opened in a such a way that it does _not_ download all objects stored within the remote Realm on the server. Instead, a partially synced Realm allows you to specify what subset of objects you want synchronized to the local copy using queries.
* **Full synchronization** will automatically synchronize the entire Realm in the background as long as the Realm is open.

The [Default Synced Realm](opening-a-synced-realm.md#the-default-synced-realm) is automatically created for you by the server and is the recommended way to get started using Query-based synchronization.

For all other Realms, the current default behavior is _Full Synchronization_ as described in the [Manually Configuring Synced Realms](opening-a-synced-realm.md#manually-configuring-synced-realms). You can choose to manually enable Query-based sync on other Realms as well, as described [here](opening-a-synced-realm.md#partial-synchronization).

For more information on how to setup a synchronized Realm and open it see the following sections:

{% page-ref page="setting-up-your-realms.md" %}

{% page-ref page="opening-a-synced-realm.md" %}

## Using Query-based synchronization

A partially synced Realm will contain no objects upon initially being created and opened, but data will be synchronized when [subscribed](syncing-data.md#subscribing-to-data) to.

### Subscribing to data

By default, a partially synchronized Realm contains no data. Instead, the client application must choose, or subscribe to, which data it wants to partially synchronize from the overall Realm maintained on the server.

Subscribing to data is easy as it utilizes Realm's query system. Applications can create any number of queries for data which will be transmitted to the server and evaluated. The results of the query will then be synced to the application. The underlying sync protocol ensures that objects are only sent once in the case that an object matches several queries an application has subscribed to.

Subscriptions are automatically persisted and maintained by the server. When data changes occur the server will reevaluate existing subscriptions and push the changes to all clients.

{% tabs %}
{% tab title="Swift" %}
```swift
let results = realm.objects(Person.self).filter("age > 18")
let subscription = results.subscribe()
```

`subscribe()` registers the query with the server, and returns a `SyncSubscription` object which can be used to observe the current state of the subscription or to remove it. A subscription can also be given an explicit name by passing the desired name to `subscribe()`:

```swift
let subscription = results.subscribe(named: "my-subscription")
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMSyncSubscription *subscription =
    [[Person objectsInRealm:realm where:@"age > 17"] subscribe];
```

`subscribe` registers the query with the server, and returns an`RLMSyncSubscription` object which can be used to observe the current state of the subscription or to remove it. A subscription can also be given an explicit name by passing the desired name to `subscribeWithName:`:

```objectivec
RLMSyncSubscription *subscription =[[Person objectsInRealm:realm where:@"age > 17"]
                                    subscribeWithName:@"my-subscription"];
```
{% endtab %}

{% tab title="Java" %}
```java
realm.where(Person.class)
  .greaterThanOrEqual("age", 18)
  .findAllAsync();
```

All asynchronous queries will automatically turn into subscriptions. Subscriptions created this way are _anonymous_ subscriptions. These kind of subscriptions cannot be unregistered again and are always running.

It is also possible to create _named_ subscriptions that can be [unregistered](syncing-data.md#unsubscribing) again.

```java
realm.where(Person.class)
  .greaterThanOrEqual("age", 18)
  .findAllAsync("my-subscription");
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
let results = realm.objects('Person').filtered('age >= 18');
let subscription = results.subscribe();
```
{% endtab %}

{% tab title=".Net" %}
```csharp
var subscription = realm.All<Person>().Where(p => p.Age > 18).Subscribe();
```

`Subscribe()` registers the query with the server, and returns a `Subscription` object which can be used to observe the current state of the subscription or to remove it. A subscription can also be given an explicit name by passing the desired name to `Subscribe()`:

```csharp
var subscription = realm.All<Person>().Where(p => p.Age > 18).Subscribe("my-subscription");
```
{% endtab %}
{% endtabs %}

### Registering for notifications

Working with a synced Realm is no different than a local Realm \(other than it is automatically synchronized\). As a result, you can utilize Realm's existing notification functionality to register for changes that occur through synchronization.

When you are using Query-based synchronization, the notification system is critical because you will need to know when the server has fulfilled the initial subscription in addition to later data changes.

{% tabs %}
{% tab title="Swift" %}
```swift
let results = realm.objects(Person.self).filter("age > 18")
let subscriptionToken = results.subscribe().observe(\.state) { state in
    switch state {
    case .creating:
        // The subscription has not yet been written to the Realm
    case .pending:
        // The subscription has been written to the Realm and is waiting
        // to be processed by the server
    case .complete:
        // The subscription has been processed by the server and all objects
        // matching the query are in the local Realm
    case .invalidated:
        // The subscription has been removed
    case .error(let err):
        // An error occurred while processing the subscription
}

let resultsToken = results.observe() { changes in
    // Called whenever the objects in the local Realm which match the query
    // change, including when a subscription being added or removed changes
    // which objects are included.
}
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMResults *results = [Person objectsInRealm:realm where:@"age > 17"];
RLMSyncSubscription *subscription = [results subscribe];

RLMNotificationToken *token = [results addNotificationBlock:^(RLMResults<Person *> *results, RLMCollectionChange *changes, NSError *error) {
     switch (subscription.state)
      {
          case RLMSyncSubscriptionStateCreating:
               // The subscription has not yet been written to the Realm
               break;
          case RLMSyncSubscriptionStatePending
               // The subscription has been written to the Realm and is waiting
               // to be processed by the server
               break;
          case RLMSyncSubscriptionStateComplete
               // The subscription has been processed by the server and all objects
               // matching the query are in the local Realm
               break;
          case RLMSyncSubscriptionStateInvalidated
               // The subscription has been removed
               break;
          case RLMSyncSubscriptionStateError:
               // An error occurred while processing the subscription
               break;
          default:
               break;
      }
}];
```
{% endtab %}

{% tab title="Java" %}
```java
RealmResults<Person> persons = realm.where(Person.class).greaterThanOrEqual("age", 18).findAllAsync();
query.addChangeListener(new OrderedRealmCollectionChangeListener<RealmResults<Person>>() {
    @Override
    public void onChange(RealmResults<Person> persons, OrderedCollectionChangeSet changeSet) {
        if (changeSet.isCompleteResult()) {
          // If true, data has been downloaded from the server.
          // If false, the query result is only based on local
          // data.
        }
    }
});
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
let results = realm.objects('Person').filtered('age >= 18');
let subscription = results.subscribe();
results.addListener((collection, changes) => {
    switch (state) {
    case Realm.Sync.SubscriptyionState.Creating
        // The subscription has not yet been written to the Realm
        break;
    case Realm.Sync.SubscriptionState.Pending
        // The subscription has been written to the Realm and is waiting
        // to be processed by the server
        break;
    case Realm.Sync.SubscriptionState.Complete:
        // The subscription has been processed by the server and all objects
        // matching the query are in the local Realm
        break;
    case Realm.Sync.SubscriptionState.Invalidated
        // The subscription has been removed
        break;
    case Realm.Sync.SubscriptionState.Error:
        console.log('An error occurred: ', subscription.error);
        break;
    }
});
```
{% endtab %}

{% tab title=".Net" %}
```csharp
var subscription = realm.All<Person>().Where(p => p.Age > 18).Subscribe();

// The Subscription class implements INotifyPropertyChanged so you can
// pass it directly to the data-binding engine
subscription.PropertyChanged += (s, e) =>
{
    switch (subscription.State)
    {
        case SubscriptionState.Creating:
            // The subscription has not yet been written to the Realm
            break;
        case SubscriptionState.Pending:
            // The subscription has been written to the Realm and is waiting
            // to be processed by the server
            break;
        case SubscriptionState.Complete:
            // The subscription has been processed by the server and all objects
            // matching the query are in the local Realm
            break;
        case SubscriptionState.Invalidated:
            // The subscription has been removed
            break;
        case SubscriptionState.Error:
            // An error occurred while processing the subscription
            var error = subscription.Error;
            break;
    }
};

subscription.Results.CollectionChanged += (s, e) =>
{
    // Called whenever the objects in the local Realm which match the query
    // change, including when a subscription being added or removed changes
    // which objects are included.
};
```
{% endtab %}
{% endtabs %}

### Unsubscribing

If a client does not need the data from a subscription anymore, it can choose to unsubscribe. This _**does not delete the objects**_, but instead simply removes them from the client device. For example, if an application provides user-driven search \(which creates a subscription\), it might choose to keep only the results from the last query to limit data stored on the device.

{% tabs %}
{% tab title="Swift" %}
```swift
let subscription = realm.objects(Person.self).filter("age > 18").subscribe()
subscription.unsubscribe()
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMSyncSubscription *subscription =
    [[Person objectsInRealm:realm where:@"age > 17"] subscribe];
[subscription unsubscribe];
```
{% endtab %}

{% tab title="Java" %}
```java
realm.unsubscribeAsync("my-subscription", new Realm.UnsubscribeCallback() {
    @Override
    public void onSuccess(String subscriptionName) {
        // Succesfully unsubscribed
    }

    @Override
    public void onError(String subscriptionName, Throwable error) {
        // Something went wrong when unsubscribing
    }
});
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
results.unsubscribe();
```
{% endtab %}

{% tab title=".Net" %}
```csharp
var subscription = realm.All<Person>().Where(p => p.Age > 18).Subscribe();
await subscription.UnsubscribeAsync();

// Alternatively, for named subscriptions, you can also unsubscribe
// by their name:
var named = realm.All<Person>().Where(p => p.Age > 18).Subscribe("legal-drivers");

await Subscription.UnsubscribeAsync("legal-drivers");
```
{% endtab %}
{% endtabs %}

## Full Synchronization

When you open a Realm without Query-based sync, the entire Realm contents will be kept in sync. You will not need to use the subscription APIs as described [above](syncing-data.md#using-partial-synchronization). Instead, to control what data is synchronized to a client, you will need to split your data up into individual Realms. For example, a common pattern is to use a global Realm for shared data across all data at a base path `/globalRealm` and user-specific data in a Realm at the user's scoped path: `/~/myRealm`.

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3)

