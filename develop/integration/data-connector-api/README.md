# Data Connector API

The Standard Edition of the Realm Platform offers a Node.js-based adapter API that allows you to access all low-level Object Server operations and data. This can be used to let a synced Realm interact with an existing legacy database such as PostgreSQL: the Realm will _also_ be kept in sync with the external database in real time. Client applications can use the Realm Database API and get the benefits of working with real-time, native objects.

The Adapter API is set up in a very similar fashion to the Event Handler API described above. Create a small Node.js application by creating a directory to place the server files, then create a `package.json` for npm dependencies or use `npm init` to create it interactively.

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
        "realm": "^1.8.0"
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

As with the event handler API, you’ll need to access the Object Server with administrative privileges, and will need to get the Object Server’s admin token. 

By default, this token is created in the `/data/keys` folder within your application project:

```javascript
// The token is the value for key ADMIN_TOKEN/my-app/data/keys/admin.json
```

To use the Adapter API, the Node.js application you’re creating will act as a translator, receiving instructions from the Object Server and calling your external database’s API to read and write to it. A sample application might look like this:

```javascript
var Realm = require('realm');

var FEATURE_TOKEN = "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...";

// Unlock Enterprise Edition APIs
Realm.Sync.setFeatureToken(FEATURE_TOKEN); 

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

### Instructions

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
* `LIST_ERASE`: erase an object in the list at the given index: this removes the object
* from the list but the object will still exist in the Realm
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

For full details, including all the instruction types and the data passed to them, consult the API reference for the `Realm.Sync.Adapter` class.

A PostgreSQL data connector is already implemented, and more are on the way, including MongoDB and Microsoft SQL Server. Any data connector can be customized to your application’s specific needs. If you’re a Realm Enterprise customer, contact your representative for more information.

Not what you were looking for? [Leave Feedback](mailto:docs-feedback@realm.io)

