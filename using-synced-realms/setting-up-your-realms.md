---
description: >-
  Key to Realm Platform is the concept of a synchronized Realm. This guide
  discusses how to get started with a synced Realm.
---

# Setting Up Your Realms

## What is a Realm?

A **Realm **is an instance of a Realm Database. Realms can be_ local, in-memory_, _or synchronized_. In practice, your application works with any kind of Realm the same way. Local Realms are the standard use-case, where the Realm simply persists data locally on the device. In-memory Realms have no persistence mechanism, and are meant for temporary storage. With Realm Platform, you will be working with a synchronized Realm.

A synchronized Realm uses the Realm Platform to transparently synchronize its contents with other devices. While your application continues working with a synchronized Realm as if it's a local Realm, the data in that Realm might be updated by any device with write access to that Realm--so the Realm could represent a channel in a chat application, for instance, being updated by any user talking in that channel. Or, it could be a shopping cart, accessible only to devices owned by you.

If you're used to working with other kinds of databases, here are some things that a Realm is _not:_

* **A Realm is not a table. **Tables typically only store one kind of information: user records, email messages, and so on. But a Realm can contain multiple kinds of objects.
* **A Realm is not a schemaless document store. **Because object properties are analogous to key/value pairs, it's easy to think of a Realm as a document store, but objects in a Realm have defined schemas that support giving values defaults or marking them as required or optional.
* **A Realm is not a traditional relational database.** A Realm is an object database in which links between objects are a first-class type. The links do not require the use of key lookups and are instead analogous to pointers.

## The Default Synced Realm

{% hint style="danger" %}
This API is new to Realm Platform 3.x and is currently in beta. Refer to the [Other Synced Realms ](setting-up-your-realms.md#other-synced-realms)section for description of behavior &lt;3.x.
{% endhint %}

The easiest way to get started with Realm Platform is to use the default synchronized Realm. With Realm Platform - Cloud each cloud instance will have a single default Realm. For Self-Hosted each server installation will include a single default Realm.

For most apps, you can include all of your application data within the default synchronized Realm. Realm Platform supports query-based sync, so that you can control what data from the default synced Realm is synchronized to the client application.

In addition, to you application data, the server will also include other internal classes used in its management and the master list of authenticated users, represented by the `__User` class. The internal data is managed by permissions restricting its access--you can learn more about how the permission system works later, for now just know this internal data exists.

The purpose of the default synchronized Realm is to simplify getting started and allow your application data to link into the master list of authenticated users.

{% hint style="info" %}
You do not need to create the default synced Realm, it is automatically created for you.
{% endhint %}

Interacting with the default synced Realm has built-in APIs in the Realm SDKs:

{% tabs %}
{% tab title="Swift" %}
The default synced Realm is provided via an automatic `SyncConfiguration`, which will use the current logged in user from `SyncUser.current` and the server URL used to authenticate.

For example, to log in, then asynchronously open the default synced Realm with the current user:

```swift
SyncUser.logIn(with: credentials, server: serverURL) { user, error in
    if let user = user {
        Realm.Configuration.defaultConfiguration = SyncConfiguration.automatic()
        Realm.asyncOpen() { realm in
            // ...
        }
    }
}
```

If you are working with multiple users, you can pass in the specific user as well:

```swift
SyncUser.logIn(with: credentials, server: serverURL) { user, error in
    if let user = user {
        Realm.Configuration.defaultConfiguration = SyncConfiguration.automatic(user: user)
        Realm.asyncOpen() { realm in
            // ...
        }
    }
}
```
{% endtab %}

{% tab title="Objective-C" %}
The default synced Realm is provided via an automatic `RLMSyncConfiguration`, which will use the current logged in user from `[RLMSyncUser currentUser]` and the server URL used to authenticate.

For example, to log in, then asynchronously open the default synced Realm with the current user:

```objectivec
[RLMSyncUser logInWithCredentials:credentials
                    authServerURL:serverURL
                     onCompletion:^(RLMSyncUser *user, NSError *error) {
    if (user) {
        RLMRealmConfiguration *config = [RLMSyncConfiguration automaticConfiguration];
        [RLMRealm asyncOpenWithConfiguration:config
                       callbackQueue:dispatch_get_main_queue()
                            callback:^(RLMRealm *realm, NSError *error) {
            if (realm) {
                // ...
            }
        }];
    }
}];
```

If you are working with multiple users, you can pass in the specific user as well:

```objectivec
[RLMSyncUser logInWithCredentials:credentials
                    authServerURL:serverURL
                     onCompletion:^(RLMSyncUser *user, NSError *error) {
    if (user) {
        RLMRealmConfiguration *config = [RLMSyncConfiguration automaticConfigurationForUser:user];
        [RLMRealm asyncOpenWithConfiguration:config
                       callbackQueue:dispatch_get_main_queue()
                            callback:^(RLMRealm *realm, NSError *error) {
            if (realm) {
                // ...
            }
        }];
    }
}];
```
{% endtab %}

{% tab title="Java" %}
The default synced Realm is provided via an automatic SyncConfiguration, which will use the current logged in user from `SyncUser.currentUser()` and the server URL used to authenticate.

For example, to log in, then asynchronously open the default synced Realm with the current user:

```java
SyncCredentials credentials = getCredentials();
String url = getUrl();
SyncUser.login(credentials, url, new SyncUser.Callback<SyncUser>() {
  @Override
  public void onSuccess(SyncUser user) {
    SyncConfiguration config = SyncConfiguration.automatic();
    Realm realm = Realm.getInstance(config);
    // Use Realm
  }
  
  @Override
  public void onError(ObjectServerError error) {
    // Handle error
  }
});
```

If you are working with multiple users, you can pass in the specific user as well:

```java
SyncUser user = getUser();
SyncConfiguration config = SyncConfiguration.automatic(user);
Realm realm = Realm.getInstance(config);
```
{% endtab %}

{% tab title="Javascript" %}
The default synced Realm is provided via the default configuration which will use the current logged in user from `Realm.Sync.User.current` and the server URL used to authenticate.

```javascript
Realm.Sync.User.login(server, username, password)
.then((user) => {
      let config = Realm.automaticSyncConfiguration();
      Realm.open(config).then((realm) => {
          // ...
      });
})
```

If you are working with multiple users, you can pass in the specific user as well:

```javascript
Realm.Sync.User.login(server, username, password)
.then((user) => {
      let config = Realm.automaticSyncConfiguration(user);
      Realm.open(config).then((realm) => {
          // ...
      });
})
```
{% endtab %}

{% tab title=".Net" %}
{% hint style="warning" %}
_API Coming Soon!_
{% endhint %}
{% endtab %}
{% endtabs %}

## Other Synced Realms

With Realm Platform, you are not restricted to only using the default synced Realm. Instead, you can create as many Realms as needed for your application. Prior to Realm Platform 3.0 and the introduction of query-based sync, using multiple Realms was the recommended pattern.

A common use-case for multiple Realms is to isolate data that is user-specific. A hypothetical chat application might use one synchronized Realm for public chats, another synchronized Realm storing user data, yet another synchronized Realm for a "master channel list" that's read-only to non-administrative users, and a local Realm for persisted settings on that device. Realms are lightweight, and your application can be using several at one time. \(On mobile platforms, there are some resource constraints, but up to a dozen open at once should be no issue.\)

The downside of multiple Realms is that objects in one Realm cannot link to objects in another, nor are cross-Realm queries supported. This means that when designing an application with multiple Realms, you must consider how your data is structured across the Realms and use keys to refer to objects in other Realms.

## Creating Models

Whether you use the default synced Realm or other Realms, the first step when using a Realm is to define the object models, or schema, for the objects it will include. With Realm Platform, you can choose to create your models for the Realm in one of two ways:

### In Your Application

The simplest way to get started with a synced Realm is to create your models in your client application directly. Realm provides language-specific SDKs that allow you to define a model as a regular class with regular properties.

{% tabs %}
{% tab title="Swift" %}
```swift
import RealmSwift

// Dog model
class Dog: Object {
    @objc dynamic var name = ""
    @objc dynamic var owner: Person? // Properties can be optional
}

// Person model
class Person: Object {
    @objc dynamic var name = ""
    @objc dynamic var birthdate = Date(timeIntervalSince1970: 1)
    let dogs = List<Dog>()
}
```

See [Models](https://realm.io/docs/swift/latest/#models) in the Swift documentation for more information on supported data types and other related concepts.
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
#import <Realm/Realm.h>

@class Person;

// Dog model
@interface Dog : RLMObject
@property NSString *name;
@property Person   *owner;
@end
RLM_ARRAY_TYPE(Dog) // define RLMArray<Dog>

// Person model
@interface Person : RLMObject
@property NSString             *name;
@property NSDate               *birthdate;
@property RLMArray<Dog *><Dog> *dogs;
@end
RLM_ARRAY_TYPE(Person) // define RLMArray<Person>

// Implementations
@implementation Dog
@end // none needed

@implementation Person
@end // none needed
```
{% endtab %}

{% tab title="Java" %}
```java
public class Dog extends RealmObject {
    private String name;
    private int age;

    // ... Generated getters and setters ...
}

public class Person extends RealmObject {
    @PrimaryKey
    private long id;
    private String name;
    private RealmList<Dog> dogs; // Declare one-to-many relationships

    // ... Generated getters and setters ...
}
```

See [Models](https://realm.io/docs/java/latest/#models) in the Java documentation for more information on supported data types and other related concepts.
{% endtab %}

{% tab title="Javascript" %}
```javascript
const Realm = require('realm');

// Define your models and their properties
const CarSchema = {
  name: 'Car',
  properties: {
    make:  'string',
    model: 'string',
    miles: {type: 'int', default: 0},
  }
};
const PersonSchema = {
  name: 'Person',
  properties: {
    name:     'string',
    birthday: 'date',
    cars:     'Car[]',
    picture:  'data?' // optional property
  }
};
```

See [Models](https://realm.io/docs/javascript/latest/#models) in the Javascript documentation for more information on supported data types and other related concepts.
{% endtab %}

{% tab title=".Net" %}
```csharp
// Define your models like regular C# classes
public class Dog : RealmObject 
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Person Owner { get; set; }
}

public class Person : RealmObject 
{
    public string Name { get; set; }
    public IList<Dog> Dogs { get; } 
}
```

See [Models](https://realm.io/docs/dotnet/latest/#models) in the .Net documentation for more information on supported data types and other related concepts.
{% endtab %}
{% endtabs %}

In JavaScript, the schema for your model must be passed into the constructor as part of the configuration object. For Swift, Java, and .Net the classes defined in your application code will automatically be added to the Realm when you open it, or you can choose to define a subset of classes for the Realm within its configuration.

### In Realm Studio

When working with a larger team or when building a cross-platform application, it can be easier to create your models in a centralized place. Realm Platform offers a desktop application, [Realm Studio](../realm-studio/), where you can connect to your cloud instance or self-hosted server and create a new synchronized Realm. You can then create new classes and add properties to the Realm.

Once you are finished creating the Realm in Studio, you can export the model definitions to any supported platform language so you can include the code in your application.

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

