# The Realm Data Model

## What is a Realm? {#what-is-a-realm}

A **Realm** is an instance of a Realm Mobile Database container. Realms can be _local, synchronized,_ or _in-memory._ In practice, your application works with any kind of Realm the same way. In-memory Realms have no persistence mechanism, and are meant for temporary storage. A synchronized Realm uses the Realm Object Server to transparently synchronize its contents with other devices. While your application continues working with a synchronized Realm as if it's a local file, the data in that Realm might be updated by any device with write access to that Realm--so the Realm could represent a channel in a chat application, for instance, being updated by any user talking in that channel. Or, it could be a shopping cart, accessible only to devices owned by you.

If you're used to working with other kinds of databases, here are some things that a Realm is _not:_

* **A Realm is not a single application-wide database.** While an application generally only uses one SQL database, an application often uses multiples Realms to organize data more efficiently, or to "silo" data for access control purposes.
* **A Realm is not a table.** Tables typically only store one kind of information: user records, email messages, and so on. But a Realm can contain multiple kinds of objects.
* **A Realm is not a schemaless document store.** Because object properties are analogous to key/value pairs, it's easy to _think_ of a Realm as a document store, but objects in a Realm have defined schemas that support giving values defaults or marking them as required or optional.

The hypothetical chat application above might use one synchronized Realm for public chats, another synchronized Realm storing user data, yet another synchronized Realm for a "master channel list" that's read-only to non-administrative users, and a local Realm for persisted settings on that device. Or a multi-user application on the same device could store each user's private data in a user-specific Realm. Realms are lightweight, and your application can be using several at one time. \(On mobile platforms, there are some resource constraints, but up to a dozen open at once should be no issue.\)

The concepts being discussed are cross-platform; simple examples will be given in Swift. Consult the documentation section for your preferred binding for examples in your language.

## Opening a Realm {#opening-a-realm}

{% hint style="info" %}
In JavaScript, the [schema](https://docs.realm.io/platform/learn/advanced/the-realm-data-model#models-and-schema) for your model must be passed into the constructor as part of the configuration object. See [Models](https://realm.io/docs/javascript/latest/#models) in the JavaScript documentation for details.
{% endhint %}

When you open a Realm, you pass the constructor a _configuration object_ that defines how to access it. The configuration object specifies where the Realm database is located:

* a path on the device's local file system
* a URL to a Realm Object Server, with appropriate access credentials \(user/password, authentication token\)
* an identifier for an in-memory Realm

\(The configuration object may specify other values, depending on your language, and is usually used for [migrations](https://realm.io/docs/data-model/#migrations) when those are necessary. As noted above, the configuration object also includes the model schema in JavaScript.\) If you don't provide a configuration object, you'll open the _default Realm,_ which is a local Realm specific to that application.

Opening a synchronized Realm, therefore, might look like this. For this example, we'll assume the Realm is named `"settings"`.

```text
// create a configuration object
let realmUrl = URL(string: "realms://example.com:9000/~/settings")!
let realmUser = SyncCredentials.usernamePassword(username: username, password: password)
let config = Realm.Configuration(user: realmUser, realmURL: realmUrl)

// open the Realm with the configuration object
let settingsRealm = try! Realm(configuration: config)
```

Opening a local or in-memory Realm is even simpler---it doesn't need a URL or user argument--and opening the default Realm is just one line:

```text
let defaultRealm = try! Realm()
```

### Realm URLs {#realm-urls}

Synchronized Realms may be _public, private,_ or _shared._ They're all accessed the same way---on a low level, there's no difference between them at all. The difference between them is [access controls](#permissions), which users can read and write to them. The URL format may also look a little different:

* A _public_ Realm can be accessed by all users. Public realms are owned by the admin user on the Realm Object Server, and are read-only to non-admins. These Realms have URLs of the form `realms://server/realm-name`.
* A _private_ Realm is created and owned by a user, and by default only that user has read and write permissions for it. Private Realms have URLs of the form `realms://server/user-id/realm-name`.
* A _shared_ Realm is a private Realm whose owner has granted other users read \(and possibly write\) access---for instance, a shopping list shared by multiple family members. It has the same URL format as a private Realm \(`realms://server/user-id/realm-name`\); the `user-id` segment of the path is the ID of the owning user. Sharing users all have their own local copies of the Realm, but there's only one "master" copy synced through the Object Server.

Very often in private Realm URLs, you'll see a tilde \(`~`\) in place of the user ID; this is a shorthand for "fill in the current user's ID." This makes it easier for application developers to refer to private Realms in code: you can simply refer to a private settings Realm, for example, with `realms://server/~/settings`.

You can think of Realm URLs as matching a file system: public Realms live at the top-level "root" directory, with user-owned Realms in subdirectories underneath. \(The tilde was chosen to match the Unix style of referring to a user's home directory with `~`.\)

{% hint style="info" %}
Note_:_ the `realms://` prefix is analogous to `https://`, e.g., the "s" indicates the use of SSL encryption. A Realm URL beginning with `realm://` is unencrypted.
{% endhint %}

### Permissions {#permissions}

Realms managed by the Realm Object Server have _access permissions_ that control whether it's public, private, or shared. Permissions are set on each Realm using three boolean flags:

* The `mayRead` flag indicates a user can read from the Realm.
* The `mayWrite` flag indicates a user can write to the Realm.
* The `mayManage` flag indicates a user can change permissions on the Realm for other users.

The permission flags can be set on a _default_ basis and a _per-user_ basis. When a user requests access for a Realm, first the Object Server checks to see if there are per-user permissions set for that user on that Realm. If there are no per-user permissions set for that user, the default permissions for the Realm are used. For example, a Realm might have `mayRead` set true by default, with individual users being granted `mayWrite` permissions.

By default, a Realm is private: the owner has all permissions on it, and no other user has _any_ permissions for it. Other users must be explicitly granted access. \(Admin users, though, are always granted all permissions to all Realms on the Object Server.\)

For more details about permissions, see:

* **Access Control** for your language SDK:

  * [Java](https://realm.io/docs/java/latest/#access-control)
  * [Objective-C](https://realm.io/docs/objc/latest/#access-control)
  * [Swift](https://realm.io/docs/swift/latest/#access-control)
  * [JavaScript](https://realm.io/docs/javascript/latest/#access-control)
  * [Xamarin](https://realm.io/docs/xamarin/latest/#access-control)

## Models and Schema {#models-and-schema}

To store an object in a traditional relational database, the object's class \(say, `User`\) corresponds to a table \(`users`\), with each object instance being mapped to a table row and the object's properties mapping to table columns. In Realm, though, your code works with the actual objects.

```text
class Dog: Object {
    dynamic var name = ""
    dynamic var age = 0
    dynamic var breed: String? = nil
    dynamic var owner: Person?
}
```

Our `Dog` object has four properties, two of which are required and have default values \(`name`, with a default of the empty string, and `age`, with a default of `0`\). The `breed` property is an optional string, and the `owner`property is an optional `Person` object. \(We'll get to that.\) Optional properties are sometimes called _nullable properties_ by Realm, meaning that their values can be set to `nil` \(or `null`, depending on your language\). Optional properties don't have to be set on objects to be stored in a Realm. Required properties, like `name` and `age`, cannot be set to `nil`.Check your language SDK for the proper syntax for required and optional properties! In Java, all properties are nullable by default, and required properties must be given the \`@Required\` notation. JavaScript marks property types and default values in a different fashion.

### Persistence and Live Objects {#persistence-and-live-objects}

There's a few things to keep in mind when working with Realms and objects in Realms.

* Once an object has been added to a Realm \(e.g., using `realm.add` or your language's equivalent\), modifying that object in your code modifies it in the Realm, too. You don't need to call an update method or re-add it to the Realm to persist your changes. They'll be updated correctly.
* This is true for synchronized Realms, too---there's nothing you need to do to "push" changes up to the Object Server. Modify the object, and when your device has network access, changes will be propagated up to the server and to any other Realm clients that are synchronizing with those Realms.

### Relations {#relations}

In relational databases, relations between tables are defined with primary keys and foreign keys. If one or more `Dog`s can be owned by one `Person`, then the `Dog` model will have a foreign key field that contains the primary key of the `Person` who owns them. This can be described as a "has-many" relationship: a `Person has-many Dogs`. The inverse relationship, `Dog belongs-to Person`, isn't explicitly defined in the database, although some ORMs implement it.

Realm has similar relationship concepts, declared with properties in your model schema.

#### To-One Relations 

Let's revisit the `Dog` model above:

```text
class Dog: Object {
    dynamic var name = ""
    dynamic var age = 0
    dynamic var breed: String? = nil
    dynamic var owner: Person?
}
```

The `owner` property is the `Object` subclass you want to establish a relationship with. This is all you need to define a "to-one" relationship \(which could be either one-to-one or many-to-one\). Now, you can define a relationship between a `Dog` and a `Person`:

```text
let bob = Person()
let fido = Dog()
fido.owner = bob
```

This is similar in other languages. Here's the Java implementation of `Dog`:

```text
public class Dog extends Realm Object {
	@Required
	private String name = "";
	
	@Required
	private Integer age = 0;
	
	private String breed;
	private Person owner;
}
```

#### To-Many Relationships

Let's show the matching `Person` class for `Dog`:

```text
class Person: Object {
	let name = ""
	let dogs = List<Dog>()
}
```

A _list_ in Realm contains one or more Realm objects. To add Fido to Bob's list of dogs:

```text
bob.dogs.append(fido)
```

Again, this is similar in other languages; in Java, you use `RealmList` as the property type \(to distinguish them from native Java lists\):

```text
public class Person extends RealmObject {
	@Required
	private String name;
	
	private RealmList<Dog> dogs;
```

{% hint style="info" %}
Note that in Java, `RealmList` properties are always considered required, so they don't need the `@Required` notation. JavaScript defines [list properties](https://realm.io/docs/javascript/latest/#list-properties%29) in a different fashion. As always, consult the documentation and API reference for your language SDKs for specifics.
{% endhint %}

#### Inverse Relationships

You'll note that defining the `Person has-many Dogs` relationship didn't automatically create a `Dog belongs-to Person` relationship; both sides of the relationship need to be set explicitly. Adding a `Dog` to a `Person`'s `dogs` list doesn't automatically set the dog's `owner`property. It's important to define both sides of this relationship: while it makes it easier for your code to traverse relationships, it's also necessary for Realm's notification system to work properly.

Some Realm language bindings provide "linking objects" properties, which return all objects that link to a given object from a specific property. To define `Dog` this way, our model could be:

```text
class Dog: Object {
    dynamic var name = ""
    dynamic var age = 0
    dynamic var breed: String? = nil
    let owners = LinkingObjects(fromType: Person.self, property: "dogs")
}
```

Now, when we execute `bob.dogs.append(fido)`, then `fido.owner`will point to `bob`.

Currently, Objective-C, Swift, and Xamarin provide linking objects.

### Primary Keys {#primary-keys}

While Realm doesn't have foreign keys, it _does_ support primary key properties on Realm objects. Declaring one of the properties on a model class to be a primary key enforces uniqueness: only one object of that class with the same primary key can be added to a Realm. Primary keys are also implicit [indexes](#indexes): querying an object on its primary key is extremely efficient.

For details about how to specify a primary key, consult your language SDK's documentation:

* [Java](https://realm.io/docs/java/latest/#primary-keys)
* [Objective-C](https://realm.io/docs/objc/latest/#primary-keys)
* [Swift](https://realm.io/docs/swift/latest/#primary-keys)
* [JavaScript](https://realm.io/docs/javascript/latest/#primary-keys)
* [Xamarin](https://realm.io/docs/xamarin/latest/#primary-keys)

Primary keys also let Realm perform an "upsert" operation: if an object is added to a Realm with a new primary key, it will be inserted, but if the key exists, adding the object with the `update` flag will _merge_ the object properties. That is, properties in the newly-added version of the object will overwrite the values of that property on the server, but any other properties on the server's copy of the object will retain their values, and the newly merged object will be downloaded to the client. Suppose you have a preference object called `Settings`, similar to a previous example:

```text
class Settings: object {
	dynamic var id = 0
	dynamic var showToolbar = true
	dynamic var linesShown = 5
}

// open Realm with an existing configuration object
let settingsRealm = try! Realm(configuration: config)
```

Suppose every `Settings` object uses the user ID as a unique primary key on the Realm Object Server. To create _or_ update a single property on a user's `Settings`:

```text
try! realm.write {
	realm.create(Settings.self, value: ["showToolbar": false], update: true)
}
```

Note that you should use the `create()` method to create the new object here. Why? If the object being added didn't specify `showToolbar` it would inherit the default value of `false`, but Realm needs to know whether this is being explicitly set and should override a value of `true` on the server. By using the `create` method, we make this choice explicit.

### Indexes {#indexes}

Adding an _index_ to a property significantly speeds up some queries. If you're frequently making an equality comparison on a property---that is, retrieving an object with an exact match, like an email address---adding an index may be a good idea. Indexes also speed up exact matches with "contains" operators \(e.g., `name IN {'Bob', 'Agatha', 'Fred'}`\).

Consult your language SDK for information on how to set indexes:

* [Java](https://realm.io/docs/java/latest/#indexing-properties)
* [Objective-C](https://realm.io/docs/objc/latest/#indexed-properties)
* [Swift](https://realm.io/docs/swift/latest/#indexed-properties)
* [JavaScript](https://realm.io/docs/javascript/latest/#indexed-properties)
* [Xamarin](https://realm.io/docs/xamarin/latest/#indexed-properties)

## Migrations {#migrations}

Since data models in Realm are defined as standard classes, making model changes is very easy. Suppose you had a `Person` model which contained these properties:

```text
class Person: Object {
	dynamic var firstName = ""
	dynamic var lastName = ""
	dynamic var age = 0
}
```

And you wished to combine the `firstName` and `lastName` properties into a single `name` property. The model change is simple:

```text
class Person: Object {
	dynamic var name = ""
	dynamic var age = 0
}
```

However, now the schema in the Realm file doesn't match your model---when you try to use the existing Realm, errors will happen. To fix this, you'll need to perform a migration---essentially, call a small bit of code in your application which can detect the old version of the Realm schema and upgrade it on disk.

### Migrations With Local Realms {#migrations-with-local-realms}

The specifics of how to perform a migration vary from language to language, but the basics are always similar:

* When you open the Realm, a schema version and migration function are passed to the constructor in the configuration object.
* If the existing schema version is equal to the schema version passed to the constructor, things proceed as normal \(no migration is performed\).
* If the existing schema version is _less_ than the schema version passed to the constructor, the migration function is called.

In Swift, a migration function added to the configuration object might look like this.

```text
let config = Realm.Configuration(
	schemaVersion: 1,
	migrationBlock: { migration, oldSchemaVersion in
		if (oldSchemaVersion < 1) {
			migration.enumerateObjects(ofType: Person.className()) { oldObject, newObject in
				let firstName = oldObject!["firstName"] as! String
				let lastName = oldObject!["lastName"] as! String
				newObject!["name"] = "\(firstName) \(lastName)"
			}
		}
	})
```

If no version is specified for a schema, it defaults to `0`.

### Migrations With Synced Realms {#migrations-with-synced-realms}

If a Realm is synced, the rules for performing migrations are a little different.

* Additive changes, such as adding a class or adding a property to an existing class, are applied automatically.
* Removing a property from a schema will not delete the field from the database, but rather instruct Realm to ignore that property. New objects will continue be created with those properties, but they will be set to `null`. Non-nullable fields will be set to appropriate zero/empty values: `0` for numeric fields, an empty string for string properties, and so on.
* Custom migration functions cannot be invoked on synced Realm migrations, and supplying one will throw an exception.
* Destructive changes---that is, changes to a Realm schema that require changes to be made to code that interacts with that Realm---are not directly supported. This includes changes to property types that keep the same name \(e.g., changing from a nullable string to a non-nullable string\), changing a primary key, or changing a field from optional to required \(or vice-versa\).

If you can't use a custom migration function, how do you make schema changes like the previous example on a synced Realm? There are two ways to do it:

* On the _client_ side, you can write a notification handler that performs the changes.
* On the _server_ side, you can write a Node.js function that performs them.

Neither of these will allow you to apply destructive changes to an existing Realm. Instead of doing that, create a new synced Realm with the new schema, then create a function which listens for changes on the old Realm and copies values to the new one. This can happen on either the client or server side.

### More about Migrations {#more-about-migrations}

To see examples and more details in your preferred language binding, consult the language-specific documentation for migrations:

* [Java](https://realm.io/docs/java/latest/#migrations)
* [Objective-C](https://realm.io/docs/objc/latest/#migrations)
* [Swift](https://realm.io/docs/swift/latest/#migrations)
* [JavaScript](https://realm.io/docs/javascript/latest/#migrations) 
* [Xamarin](https://realm.io/docs/xamarin/latest/#migrations)



Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

