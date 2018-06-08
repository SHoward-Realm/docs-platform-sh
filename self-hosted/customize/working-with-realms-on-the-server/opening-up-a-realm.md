# Opening up a Realm

## Getting Started

If you ever need to open up a realm in your `index.ts` file, you can easily request the server instance with a call to `server.openRealm`

```typescript
server.openRealm('/products')
  .then(realm => {
    console.log('My products realm!', realm)
  })
```

A schema is not required to open up a realm. However if you’d like to get a realm with specific schemas. Then you can run:

```typescript
const ProductSchema: Realm.ObjectSchema = {
  name: 'Product',
  primaryKey: 'productId',
  properties: {
    productId: 'int',
    name: 'string',
    price: 'float'
  }
}

const CompanySchema: Realm.ObjectSchema = {
  name: 'Company',
  primaryKey: 'companyId',
  properties: {
    companyId: 'int',
    name: 'string',
    address: 'string'
  }
}

server.openRealm('/products', [ProductSchema, CompanySchema])
  .then(realm => {
    console.log('My products realm!', realm)
  })
```

The realm is always opened up with admin privileges.

## Opening Realms Asynchronously 

In general, work done with synchronized realms will match that of its non-synchronized counterparts.  However, there are a few noteworthy caveats.  If a user only has read-only access to a Realm, it must be obtained with `Realm.openAsync`.  When this function is called, the data is fetched from the local sync-ed realm as well as any new data from the server since the last synchronization with your local copy.  To reiterate, this is only the case if a user does not have write access to the synchronized realm in question. 

### Permission Denied Errors

The reason why you get a `"permission denied"` error if you attempt to open a read-only synced Realm synchronously is that upon initialization a local Realm file will be created.  Then immediately, write operations are performed to write the Realm's schema \(i.e. create db tables, columns, and metadata\). However, since the user does not have write access to the Realm, the Realm Object Server \(ROS\) rejects the changes and triggers your global error handler notifying you that an illegal attempt to modify the file was made by your user.

The reason why this doesn't happen with `openAsync` is that it's an asynchronous operation and therefore doesn't need to give you a valid Realm immediately, so it doesn't need to "bootstrap" it by writing the schema to it. Instead, it requests the latest state of the Realm from ROS and vends it back to you once it's fully available in its latest state at the point in time at which the call was started.

If the local copy of the Realm already has its schema initialized \(i.e. after a successful `openAsync` call\) and the in-memory schema defined by either the default schema or the custom object types specified in `Realm.Configuration` hasn't changed, then no schema will be attempted to be written to the file.

This means that any time after a successful `openAsync` call, the Realm could be accessed synchronously without going through `openAsync` .  However, you may not have the most up to date data from the ROS.



Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

