# Key Concepts

## The Realm Platform {#the-realm-platform}

The Realm Platform is made up of two major components.  The Realm Database and the Realm Object Server.  These two components work in conjunction to automatically synchronize data enabling a great deal of use cases ranging from offline apps to complex backend integrations.  

### The Realm Database {#the-realm-database}

At the heart of the Realm Platform is the **Realm Database,** an open source, embedded database library optimized for mobile use. If you've used a data store like SQLite or Core Data, at first glance the Realm Database may appear to be an object-relational mapper \(ORM\) backed with a lightweight relational database. This is not, however, what Realm is. Instead, Realm uses a "data container" model. Your data objects are stored in a Realm _as objects._ 

### The Realm Object Server {#the-realm-object-server}

Applications built on the Realm Platform have the ability to access and create Realms on the Realm Object Server. All your application needs is the connection information for your Object Server and the URLs of the server-side Realms. This means that any of the data stored in the Realm database can be automatically synchronized from mobile device to server.  

Realms always operate in an "offline first" fashion: reads and writes take place on the local copy of the Realm. Syncing is automatic and fully transactional, and takes place as soon as a data connection is available.

Want to take a deeper look? Read the [Realm Overview White Paper](https://www2.realm.io/whitepaper/realm-overview-registration), a comprehensive overview that covers core concepts, key use cases, and implementation examples.

### Take a Deeper Dive into the [Realm Data Model](../learn/advanced/the-realm-data-model.md)

## What's next?  [Install the Realm Object Server](install-realm-object-server/)  {#getting-started}

Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

