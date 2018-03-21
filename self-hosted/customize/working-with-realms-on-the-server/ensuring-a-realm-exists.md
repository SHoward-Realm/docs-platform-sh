# Ensuring a Realm Exists

If you want to make sure a Realm exists \(for example, if you want to [apply permissions](changing-permissions-of-a-server-realm.md) to it\), you can do so by calling

```javascript
server.ensureRealmExists('/my-realm')
  .then(result => {
    // The Realm file exists and is empty, you can now apply permissions to it
  });
```

If you want to create the Realm file in a user’s directory, for example as part of a bootstrapping logic upon user creation, you can pass an optional second argument that is the Realm owner’s Id:

```javascript
const ownerId = 'some-user-id';
// When ownerId is passed, you can use ~ in the url
server.ensureRealmExists('/~/my-realm', ownerId)
  .then(result => {
    // The Realm file has been created and the user has admin privileges to access it
  });
```

If you don’t specify `ownerId`, then the Realm will still be created in the user’s directory but they won’t have permissions to access it. This can be useful, for example, when you want to bootstrap a readonly per-user Realm, in which case, you’ll need to call `server.applyPermissions` before the user can access the Realm.



Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

