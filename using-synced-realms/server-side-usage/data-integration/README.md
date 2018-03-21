---
description: >-
  Realm Platform is designed to integrate into existing systems-of-record
  through a dedicated API that outputs all synced operations. Pre-built
  connectors are also available offering two-way DB sync.
---

# Data integration

## Overview

Realm Platform offers a Node.js-based Adapter API that allows you to access all low-level Object Server operations and data. This can be used to capture all the changes that come into the server and use them to replicate the data into a legacy database, such as Postgres, or feed into existing streaming infrastructure, such as Kafka.

The adapter API is designed for one-way transfer of all changes, including schema changes, to another system. To apply changes to a synchronized Realm, for two-way replication, you would use the existing SDK APIs with an admin user as described in the [data access section](../data-access.md).

### Pre-built Data Connectors

Realm offers the following pre-built database connectors that utilize the Adapter API:

* Postgres
* SQLServer

These database connectors offer two-way replication. They use the Adapter API to capture changes from Realm and convert the operations into the applicable database commands in the other system. In addition, they use different mechanisms to perform "Change Data Capture" from the other system and convert the operations into Realm API commands. As a result, the database connectors provide automatic realtime two-way synchronization simplifying integration work. For more information consult their documentation.

## How to use the Adapter API

The Adapter API is set up in a very similar fashion to the [Event Handler API.](../data-change-events.md#creating-an-event-handler) Create a small Node.js application by creating a directory to place the server files, then create a `package.json` for npm dependencies or use `npm init` to create it interactively.

{% code-tabs %}
{% code-tabs-item title="package.json" %}
```javascript
{
    "name": "MyApp",
    "version": "0.0.1",
    "main": "index.js",
    "author": "Your Name",
    "description": "My Cool Realm App",
    dependencies":{
        "realm": "^2.3.0"
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
As with the [Event Handler API](../data-change-events.md#creating-an-event-handler), you’ll need to access the Object Server with an [admin user.](../../working-with-users/admin-users.md)
{% endhint %}

To use the Adapter API, the Node.js application you’re creating will act as a translator, receiving instructions from the Object Server and calling your external database’s API to read and write to it. A sample application might look like this:

```javascript
var Realm = require('realm');

var adapterConfig = {
  // Insert the Realm admin token here
  admin_token: 'ADMIN_TOKEN',

  // the URL to the Realm Object Server
  server_url: 'realm://127.0.0.1:9080',

  // local path for the Adapter API file
  local_path: './adapter',

  // regular expression to limit which Realms will be observed
  realm_path_regex: '/^\/([0-9a-f]+)\/private$/'
};

class CustomAdapter {
  constructor(config) {
    this.adapter = new Realm.Sync.Adapter(
      config.local_path,
      config.server_url,
      Realm.Sync.User.adminUser(config.admin_token, config.server_url),
      config.realm_path_regex,

      // This callback is called any time a new transaction is available for
      // processing for the given path. The argument is the path to the Realm
      // for which changes are available. This will be called for all Realms
      // which match realm_path_regex.
      (realm_path) => {
        var current_instructions = this.adapter.current(realm_path);
        while (current_instructions) {
          // if defined, process the current array of instructions
          this.process_instructions(current_instructions);

          // call advance to progress to the next transaction
          this.adapter.advance(realm_path);
          current_instructions = this.adapter.current(realm_path);
        }
      }
    )
  }

  // This method is passed the list of instructions returned from
  // Adapter.current(path)
  process_instructions(instructions) {
    instructions.forEach((instruction) => {
        // perform an operation for each type of instruction
        switch (instruction.type) {
          case 'INSERT':
            insert_object(instruction.object_type, instruction.identity, instruction.values);
            break;
          case 'DELETE':
            delete_object(instruction.object_type, instruction.identity);
            break;
          // ... add handlers for all other relevant instruction types
          default:
            break;
        }
      })
  }
}

new CustomAdapter(adapterConfig);
```

### Available Instructions {#instructions}

Each instruction object returned by `Adapter.current` has a `type` property which is one of the following strings. Two or more other properties containing data for the instruction processing will also be set.

* `INSERT`: insert a new object
  * `object_type`: type of the object being inserted
  * `identity`: primary key value or row index for the object
  * `values`: map of property names and property values for the object to insert
* `SET`: set property values for an existing object
  * `object_type`: type of the object
  * `identity`: primary key value or row index for the object
  * `values`: map of property names and property values to update for the object
* `DELETE`: delete an existing object
  * `object_type`: type of the object
  * `identity`: primary key value or row index for the object
* `CLEAR`: delete all objects of a given type
  * `object_type`: type of the object
* `LIST_SET`: set the object at a given list index to an object
  * `object_type`: type of the object
  * `identity`: primary key for the object
  * `property`: property name for the list property to mutate
  * `list_index`: list index to set
  * `object_identity`: primary key or row number of the object being set
* `LIST_INSERT`: insert an object in the list at the given index
  * `object_type`: type of the object
  * `identity`: primary key for the object
  * `property`: property name for the list property to mutate
  * `list_index`: list index at which to insert
  * `object_identity`: primary key or row number of the object to insert
* `LIST_ERASE`: erase an object in the list at the given index: this removes the object from the list but the object will still exist in the Realm
  * `object_type`: type of the object
  * `identity`: primary key for the object
  * `property`: property name for the list property to mutate
  * `list_index`: list index which should be erased
* `LIST_CLEAR`: clear a list removing all objects: objects are not deleted from the Realm
  * `object_type`: type of the object
  * `identity`: primary key for the object
  * `property`: property name for the list property to clear
* `ADD_TYPE`: add a new type
  * `object_type`: name of the type
  * `primary_key`: name of primary key property for this type
  * `properties`: Property map as described in [Realm.ObjectSchema](https://realm.io/docs/javascript/latest/api/Realm.html#~ObjectSchema) 
* `ADD_PROPERTIES`: add properties to an existing type
  * `object_type`: name of the type
  * `properties`: Property map as described in [Realm.ObjectSchema](https://realm.io/docs/javascript/latest/api/Realm.html#~ObjectSchema) 
* `CHANGE_IDENTITY`: change the row index for an existing object: not called for objects with primary keys
  * `object_type`: type of the object
  * `identity`: old row value for the object
  * `new_identity`: new row value for the object

For full details, including all the instruction types and the data passed to them, consult the [API reference](https://realm.io/docs/javascript/2.2.0/api/Realm.Sync.Adapter.html) for the `Realm.Sync.Adapter` class.



A PostgreSQL and Microsoft SQL Server data connectors are already implemented, and more are on the way, including MongoDB. Any data connector can be customized to your application’s specific needs. If you’re a Realm Enterprise customer, contact your representative for more information.

Not what you were looking for? [Leave Feedback](mailto:docs-feedback@realm.io) 

