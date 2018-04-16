# Migrating Your Data

## Additive Changes

Synchronized Realms only support additive changes. This is to ensure that older clients can continue to sync with newer clients. As a result, synced migrations tend to be simpler.

{% hint style="info" %}
If you remove a property from a schema with a synced Realm, this only removes the reference in your code. The underlying property is not removed from the database to maintain backwards compatibility.
{% endhint %}

{% tabs %}
{% tab title="Swift" %}
When your Realm is synced with Realm Object Server, the migration process is a little different—and in many cases, simpler. Here’s what you need to know:

* You don’t need to set the schema version \(although you can\).
* Additive changes, such as adding a class or adding a field to a class, are applied automatically.
* Removing a field from a schema doesn’t delete the field from the database, but instead instructs Realm to ignore that field. New objects will continue to be created with those properties, but they will be set to `null`. Non-nullable fields will be set to appropriate zero/empty values: `0` for numeric fields, an empty string for string properties, and so on.
* You _must not_ include a migration block.

Suppose your application has the `Dog` class:

```swift
class Dog: Object {
    @objc dynamic var name = ""
}
```

Now you need to add the `Person` class and give it an `owner` relationship to `Dog`. You don’t need to do anything other than adding the class and associated properties before syncing:

```swift
class Dog: Object {
    @objc dynamic var name = ""
    @objc dynamic var owner: Person?
}

class Person: Object {
    @objc dynamic var name = ""
    @objc dynamic var birthdate: NSDate? = nil
}

let syncServerURL = URL(string: "http://localhost:9080/Dogs")!
let config = Realm.Configuration(syncConfiguration: SyncConfiguration(user: user, realmURL: syncServerURL))

let realm = try! Realm(configuration: config)
```
{% endtab %}

{% tab title="Objective-C" %}
When your Realm is synced with Realm Object Server, the migration process is a little different—and in many cases, simpler. Here’s what you need to know:

* You don’t need to set the schema version \(although you can\).
* Additive changes, such as adding a class or adding a field to a class, are applied automatically.
* Removing a field from a schema doesn’t delete the field from the database, but instead instructs Realm to ignore that field. New objects will continue to be created with those properties, but they will be set to `null`. Non-nullable fields will be set to appropriate zero/empty values: `0` for numeric fields, an empty string for string properties, and so on.
* You _must not_ include a migration block.

Suppose your application has the `Dog` class:

```objectivec
@interface Dog : RLMObject
@property NSString *name;
@end
```

Now you need to add the `Person` class and give it an `owner` relationship to `Dog`. You don’t need to do anything other than adding the class and associated properties before syncing:

```objectivec
@interface Dog : RLMObject
@property NSString *name;
@property Person   *owner;
@end
RLM_ARRAY_TYPE(Dog)

@interface Person : RLMObject
@property NSString *name;
@property NSDate   *birthdate;
@end
RLM_ARRAY_TYPE(Person)

// Required properties of an Objective‑C reference
// type have to be declared in combination with:
@implementation Person
+ (NSArray *)requiredProperties {
    return @[@"name"];
}
@end

NSURL *syncServerURL = [NSURL URLWithString:@"http://localhost:9080/Dogs"];
RLMRealmConfiguration *config = [RLMRealmConfiguration defaultConfiguration];
config.syncConfiguration = [[RLMSyncConfiguration alloc] initWithUser:user realmURL:syncServerURL];

RLMRealm *realm = [RLMRealm realmWithConfiguration:config error:nil];
```
{% endtab %}

{% tab title="Java" %}
When your Realm is synced with Realm Object Server, the migration process is a little different—and in many cases, simpler. Here’s what you need to know:

* You don’t need to set the schema version \(although you can\).
* Additive changes, such as adding a class or adding a field to a class, are applied automatically.
* Removing a field from a schema doesn’t delete the field from the database, but instead instructs Realm to ignore that field. New objects will continue to be created with those properties, but they will be set to `null`. Non-nullable fields will be set to appropriate zero/empty values: `0` for numeric fields, an empty string for string properties, and so on.
* You _must not_ include a migration block.

Suppose your application has the `Dog` class:

```java
public class Dog extends RealmObject {
    private String name = "";
}
```

Now you need to add the `Person` class and give it an `owner` relationship to `Dog`. You don’t need to do anything other than adding the class and associated properties before syncing:

```java
public class Dog extends RealmObject {
  private String name = "";
  private Person owner;
  // getter / setter
}

public class Person extends RealmObject {
  private String name = "";
  private Date birthdate;
  // getter / setter
}

String syncServerURL = "realm://localhost:9080/Dogs"
SyncConfiguration config = new SyncConfiguration.Builder(user, syncServerURL).build();

Realm realm = Realm.getInstance(config);
```
{% endtab %}

{% tab title="Javascript" %}
When your Realm is synced with Realm Object Server, the migration process is a little different—and in many cases, simpler. Here’s what you need to know:

* You don’t need to set the schema version \(although you can\).
* Additive changes, such as adding a class or adding a field to a class, are applied automatically.
* Removing a field from a schema doesn’t delete the field from the database, but instead instructs Realm to ignore that field. New objects will continue to be created with those properties, but they will be set to `null`. Non-nullable fields will be set to appropriate zero/empty values: `0` for numeric fields, an empty string for string properties, and so on.
* You _must not_ include a migration block.

Suppose your application has the `Dog` class:

```javascript
let schemas.Dog = {
  name: 'Dog',
  properties: {
    name: 'string'
  }
};
```

Now you need to add the `Person` class and give it an `owner` relationship to `Dog`. You don’t need to do anything other than adding the class and associated properties before syncing:

```javascript
let schemas.Dog = {
  name: 'Dog',
  properties: {
    name: 'string',
    owner: 'Person'
  }
};

let schemas.Person = {
  name: 'Person',
  properties: {
    name: 'string',
    birthdate: 'date'
  }
};

let config = {
  sync: {
     user: user,
     url: syncServerUrl
  },
  schema: [schemas.Person, schemas.Dog] 
};
Realm.open(config).then(realm => { /* ... */ });
```
{% endtab %}

{% tab title=".Net" %}
When your Realm is synced with Realm Object Server, the migration process is a little different—and in many cases, simpler. Here’s what you need to know:

* You don’t need to set the schema version \(although you can\).
* Additive changes, such as adding a class or adding a field to a class, are applied automatically.
* Removing a field from a schema doesn’t delete the field from the database, but instead instructs Realm to ignore that field. New objects will continue to be created with those properties, but they will be set to `null`. Non-nullable fields will be set to appropriate zero/empty values: `0` for numeric fields, an empty string for string properties, and so on.
* You _must not_ include a migration block.

Suppose your application has the `Dog` class:

```csharp
public class Dog : RealmObject
{
    public string Name { get; set; }
}
```

Now you need to add the `Person` class and give it an `Owner` relationship to `Dog`. You don’t need to do anything other than adding the class and associated properties before syncing:

```csharp
public class Dog : RealmObject
{
    public string Name { get; set; }
    public Person Owner { get; set; }
}

public class Person : RealmObject {
    public string Name { get; set; }
    public DateTimeOffset Birthdate { get; set; }
}

var syncServerUri = new Uri("http://localhost:9080/Dogs");
var config = new SyncConfiguration(user, syncServerUri);

var realm = Realm.GetInstance(config);
```
{% endtab %}
{% endtabs %}

## Destructive Changes

If you need to perform a destructive schema change, you will need to create a new Realm. You can also manually delete the existing Realm and recreate it if you are in development.

{% tabs %}
{% tab title="Swift" %}
Since synced Realms don’t support migration blocks, destructive changes for a migration—changing a primary key, changing field types of existing fields \(while keeping the same name\), or changing a property from optional to required or vice-versa—need to be handled in a different way. Create a _new_ synchronized Realm with the new schema, and copy data from the old Realm to the new Realm:

```swift
class Dog: Object {
    @objc dynamic var name = ""
    @objc dynamic var owner: Person?
}

class Person: Object {
    @objc dynamic var name = ""
}

class PersonV2: Object {
    @objc dynamic var name: String? = nil
}

var syncServerURL = URL(string: "realm://localhost:9080/Dogs")!
var config = Realm.Configuration(syncConfiguration: SyncConfiguration(user: user, realmURL: syncServerURL))
// Limit to initial object type
config.objectTypes: [Dog.self, Person.self]

let initialRealm = try! Realm(configuration: config)


syncServerURL = URL(string: "realm://localhost:9080/DogsV2")!
config = Realm.Configuration(syncConfiguration: SyncConfiguration(user: user, realmURL: syncServerURL))
// Limit to new object type
config.objectTypes: [Dog.self, PersonV2.self]

let newRealm = try! Realm(configuration: config)
```

Custom migrations can also be applied to synced Realms by writing a [notification handler](https://realm.io/docs/swift/latest/#notifications) on the client side to make the changes, or as a [server-side event handler](server-side-usage/data-change-events.md#creating-an-event-handler). However, if the migration makes a destructive change, the Realm will stop syncing with ROS, producing a `Bad changeset received` error.
{% endtab %}

{% tab title="Objective-C" %}
Since synced Realms don’t support migration blocks, destructive changes for a migration—changing a primary key, changing field types of existing fields \(while keeping the same name\), or changing a property from optional to required or vice-versa—need to be handled in a different way. Create a _new_ synchronized Realm with the new schema, and copy data from the old Realm to the new Realm:

```objectivec
@interface Dog : RLMObject
@property NSString *name;
@property Person *owner;
@end
RLM_ARRAY_TYPE(Dog)

@interface Person : RLMObject
@property NSString *name;
@end
RLM_ARRAY_TYPE(Person)

@implementation Person
+ (NSArray *)requiredProperties {
    return @[@"name"];
}

@interface PersonV2 : RLMObject
@property NSString *name;
@end
RLM_ARRAY_TYPE(PersonV2)

@implementation PersonV2
+ (NSArray *)requiredProperties {
    return @[];
}

NSURL *syncServerURL = [NSURL URLWithString: @"realm://localhost:9080/Dogs"];
RLMRealmConfiguration *config = [RLMRealmConfiguration defaultConfiguration];
config.syncConfiguration = [[RLMSyncConfiguration alloc] initWithUser:user realmURL:syncServerURL];
// Limit to initial object type
config.objectClasses = @[Dog.class, Person.class];

RLMRealm *initialRealm = [RLMRealm realmWithConfiguration:config error:nil];

syncServerURL = [NSURL URLWithString: @"realm://localhost:9080/DogsV2"];
config = [RLMRealmConfiguration defaultConfiguration];
config.syncConfiguration = [[RLMSyncConfiguration alloc] initWithUser:user realmURL:syncServerURL];
// Limit to new object types
config.objectClasses = @[Dog.class, PersonV2.class];

RLMRealm *newRealm = [RLMRealm realmWithConfiguration:config error:nil];
```
{% endtab %}

{% tab title="Java" %}
Since synced Realms don’t support migration blocks, destructive changes for a migration—changing a primary key, changing field types of existing fields \(while keeping the same name\), or changing a property from optional to required or vice-versa—need to be handled in a different way. Create a _new_ synchronized Realm with the new schema, and copy data from the old Realm to the new Realm:

```java
public class Dog extends RealmObject {
  private String name = "";
  private Person owner;
  // getter / setter
}

public class Person extends RealmObject {
  private String name = "";
  // getter / setter
}

public class DogV2 extends RealmObject {
  private String name = "";
  // getter / setter
}

@RealmModule(classes = { Person.class, Dog.class }) class InitialModule {}

String syncServerURL = "realm://localhost:9080/Dogs"
SyncConfiguration config = new SyncConfiguration.Builder(user, syncServerURL)
              .modules(new InitialModule())
              .build();
// Limit to initial object type
Realm initialRealm = Realm.getInstance(config);



@RealmModule(classes = { Person.class, DogV2.class }) class NewModule {}

String syncServerURL = "realm://localhost:9080/Dogs"
SyncConfiguration config = new SyncConfiguration.Builder(user, syncServerURL)
              .modules(new NewModule())
              .build();
// Limit to new object type
Realm initialRealm = Realm.getInstance(config);
```

Your application can do this by listening for [notifications](https://realm.io/docs/java/latest/#notifications) on the old Realm, making the changes and copying them to the new Realm. \(This is a good use case for [Dynamic Realms](https://realm.io/docs/java/latest/#dynamic-realms).\) You can also use a Realm Function \(or event handler\) running on the Object Server.
{% endtab %}

{% tab title="Javascript" %}
Since synced Realms don’t support migration blocks, destructive changes for a migration—changing a primary key, changing field types of existing fields \(while keeping the same name\), or changing a property from optional to required or vice-versa—need to be handled in a different way. Create a _new_ synchronized Realm with the new schema, and copy data from the old Realm to the new Realm:

```javascript
let schemas.Dog = {
  name: 'Dog',
  properties: {
    name: 'string',
    owner: 'Person'
  }
};

let schemas.Person = {
  name: 'Person',
  properties: {
    name: 'string',
    birthdate: 'date'
  }
};

let schemas.DogV2= {
  name: 'DogV2',
  properties: {
    name: 'string'
  }
};

let config = {
  sync: {
    user: user,
    url: 'realm://localhost:9080/Dogs'
  },
  schema: [schemas.Dog, schemas.Person]
}

let realm = new Realm(config);

let configV2 = {
  sync: {
    user: user,
    url: 'realm://localhost:9080/DogsV2'
  },
  schema: [schemas.DogV2, schemas.Person]
}

let realmV2 = new Realm(configV2);

// copy objects from realm to realmV2
```
{% endtab %}

{% tab title=".Net" %}
Since synced Realms don’t support migration blocks, destructive changes for a migration—changing a primary key, changing field types of existing fields \(while keeping the same name\), or changing a property from optional to required or vice-versa—need to be handled in a different way. Create a _new_ synchronized Realm with the new schema, and copy data from the old Realm to the new Realm:

```csharp
public class Dog : RealmObject
{
    public string Name { get; set; }
    public Person Owner { get; set; }
}

// We can't change the name of the class in Realm, so we're
// using [MapTo] to avoid collisions between the .NET classes.
[MapTo("Person")]
public class Person_Old : RealmObject
{
    // By default strings are nullable
    public string Name { get; set; }
}

public class Person : RealmObject
{
    // Changing optionality is destructive change
    [Required]
    public string Name { get; set; }
}

var initialConfig = new SyncConfiguration(user, new Uri("http://localhost:9080/Dogs"))
{
    // Limit to initial object type
    ObjectClasses = new[] { typeof(Dog), typeof(Person_Old) }
};

var initialRealm = Realm.GetInstance(initialConfig);

var newConfig = new SyncConfiguration(user, new Uri("http://localhost:9080/DogsV2"))
{
    // Limit to new object type
    ObjectClasses = new[] { typeof(Dog), typeof(Person) }
};

var newRealm = Realm.GetInstance(newConfig);
```

Custom migrations can also be applied to synced Realms by writing a [notification handler](https://realm.io/docs/dotnet/latest/#notifications) on the client side to make the changes, or as a [server-side event handler](server-side-usage/data-change-events.md#creating-an-event-handler). However, if the migration makes a destructive change, the Realm will stop syncing with ROS, producing a `Bad changeset received` error.
{% endtab %}
{% endtabs %}

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3)

