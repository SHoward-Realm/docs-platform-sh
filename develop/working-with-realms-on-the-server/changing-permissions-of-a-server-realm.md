# Changing Permissions of a Server Realm

If you need to grant or revoke access to a synced realm, you can do that by calling `server.applyPermissions`. This can be used for bootstrapping a server without relying on an external service to set the correct permissions of global Realm files. To be able to change the permissions of the file, it must exist.

```typescript
server.applyPermissions({ userId: '*' }, '/my-shared-realm', 'read')
  .then(result => {
    console.log('Permissions applied. Affected users: ' + result.affectedUsers);
  });
```

Permissions can be applied to global realm \(`/my-realm`\) or to user-owned realms \(`/some-user-id/my-realm`\). They can be applied either by user id, in which case `*` means all users, including users created later, or by metadata key/value pair. The supported access levels are `none`, `read`, `write`, and `admin`.



Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

