---
description: >-
  In this tutorial, we're going to see how we can use Realm's permission system
  in order to limit which users can see our projects within our ToDo
  application.
---

# Step 3 - Adding Permissions

{% hint style="danger" %}
## _This tutorial uses functionality not yet deployed on Realm Cloud._

This tutorial uses a beta version of new Realm APIs. This code is _**not yet production ready**_ and may not reflect the final API. We will be updating this tutorial as the API and the functionality are made ready for general availability.
{% endhint %}

In the previous tutorial we used partial sync to synchronize only projects owned by the current user. However, nothing prevented a malicious user from seeing another user's projects by subscribing with a query that matches a broader set of `Project` instances.

To address this issue we will instead use Realm's permission system to limit access to each `Project` instance so only the user that created it can read, modify, or delete it.

## Quick Start {#quick-start}

{% hint style="success" %}
Want to get started right away with the complete source code? [Clone the demo app repository from GitHub](https://github.com/realm/my-first-realm-app), then follow the instructions in [`ios/ObjectPermissions/README.md`](https://github.com/realm/my-first-realm-app/blob/master/ios/ObjectPermissions/README.md) to get started. Don't forget to update the `Constants.swift` file with your Realm Cloud instance URL before running the app.
{% endhint %}

## Setting up a new Realm with Realm Studio {#setting-up-a-new-realm-with-realm-studio}

Realm Studio is a powerful tool, and one of the advanced features it provides is the ability to create a Realm and assign permissions to it, without needing to do this programmatically.

### Create the reference Realm {#create-the-reference-realm}

We need to create a global Realm that will contain the `Project` and `Item` objects for all users. This is known as the reference Realm, and it will be opened by each user using partial sync.

1. Open Realm Studio and Click on _Connect to the Server._ By default Realm Studio will use the auto-generated admin user \(`realm-admin`\) to connect as an admin to your instance.
2. Create a new global Realm.
   1. Click _Create new Realm_.
   2. Enter `ToDo-permissions` for the path.
   3. Click _Create Realm_.
3. Make the `ToDo-permissions` Realm readable and writable by all authenticated users:
   1. Open the special `__admin` Realm from Realm Studio, then select `Permission.`
   2. Click on _Create new Permission_ to add a new permission for our `ToDo-permissions` Realm.
   3. Click on `realmFile` and select the `ToDo-permissions` Realm.
   4. Select `true` for `mayRead` and `mayWrite` to give read and write permission on this Realm to all authenticated users.
   5. Click on _Create_ to create the permission. This will add it to the list of permissions in the `__admin` Realm.

You can verify that new `ToDo-permissions` Realm is set up correctly by checking that its ownership is reported as `public R W`.

{% hint style="info" %}
When creating the permission we left the `user` field empty \(`null`\). This indicates that the permission will apply to all users.
{% endhint %}

## Updating the ToDo application {#updating-the-todo-application}

### Create a new role per user {#create-a-new-role-per-user}

Permissions are assigned to a collection of zero or more users named a _role_. Since we want a given `Project` to be accessible only to a single user, we will create a separate role for each user. We can then grant the current user's role the appropriate permissions when we create a new `Project`. This will have the effect of preventing all other users from seeing or modifying that `Project` instance.

To create the new role, after the user successfully logs in, we verify that we have a `PermissionRole` that corresponds to the current `SyncUser`. If the role doesn't exist, we create the role and add the user to it.

```swift
// Ensure there's a role named for the user with the user as a member.
func ensureRoleExists(_ user: SyncUser, _ realm: Realm) {
    let identity = user.identity!
    guard realm.objects(PermissionRole.self).filter("name = %@ AND ANY users.identity = %@",
                                                    identity, identity).isEmpty
    else {
        return
    }

    let permissionUser = realm.create(PermissionUser.self, value: [identity], update: true)
    let role = realm.create(PermissionRole.self, value: [identity], update: true)
    role.users.append(permissionUser)
}
```

### Control access to `Project` instances {#control-access-to-project-instances}

Next we add a `permissions` attribute to the `Project` model. This tells Realm that we wish to configure the permissions of each `Project` instance independently, and serves as the means for doing so.

```swift
class Project: Object {
    // ...
    let permissions = List<Permission>()
}
```

{% hint style="info" %}
Note that we no longer need the `owner` attribute in our `Project` class as we're relying on the permission system, rather than queries, to limit access.
{% endhint %}

The `Permission` class represents the set of privileges we want to grant and the role to which they should be granted. Since we want to limit each `Project` to being visible to only a single user, we will grant the current user's role the appropriate permissions when we create a new `Project`. This will have the result of preventing all other users from seeing or modifying the new `Project` instance.

### Set the permissions of a new `Project` {#set-the-permissions-of-a-new-project}

As we create a new `Project` we now associate a new `Permission` with it to restrict who has access to it.

```swift
let textField = alertController.textFields![0]
let project = Project()
project.name = textField.text ?? ""

try! self.realm.write {
    self.realm.add(project)

    let permission = project.permissions.findOrCreate(forRoleNamed: SyncUser.current!.identity!)
    permission.canRead = true
    permission.canUpdate = true
    permission.canDelete = true
}
```

By stating that the current user's role can read, update, and delete the `Project`, we prevent any other user from having access to the `Project`.

### Lower the default permissions {#lower-the-default-permissions}

By default Realm Cloud creates a set of permissive default permissions and roles. This allows getting started on developing your application without first having to worry about permissions and roles. However, these permissive default permissions should be tightened before deploying your application.

By default, all users are added to an `everyone` role, and this role has access to all objects within the Realm. As part of lowering the default permissions, we will limit the access of this `everyone` role.

{% hint style="warning" %}
In this demo, we lower the privileges programmatically the first time a user logs in to our ToDo application. In practice, you'll want to configure these privileges using Realm Studio or a script prior to your users running the application.
{% endhint %}

There are two levels of permissions that we will lower: class-level permissions, and Realm-level permissions.

#### Class-level permissions {#class-level-permissions}

We use permissions to limit querying to only the `Project` type. This means that the only `Item` objects that can be synchronized are those associated with a `Project` that we have permissions to access. Additionally, we remove the ability for `everyone` to change the permisisons of each of our model classes.

```swift
// Ensure that class-level permissions cannot be modified by anyone but admin users.
// The Project type can be queried, while Item cannot. This means that the only Item
// objects that will be synchronized are those associated with our Projects.
let queryable = [Project.className(): true, Item.className(): false]
for cls in [Project.self, Item.self] {
    let classPermissions = realm.objects(ClassPermission.self).filter("name = %@", cls.className()).first!
    let everyonePermission = classPermissions.permissions.findOrCreate(forRoleNamed: "everyone")
    everyonePermission.canQuery = queryable[cls.className()]!
    everyonePermission.canSetPermissions = false
}
```

#### Realm-level permissions {#realm-level-permissions}

We use permissions to prevent the schema and permissions from being modified. Note that the order in which we perform these operations is significant, as removing the ability to modify permissions would cause subsequent permission changes to be rejected.

```swift
// Ensure that the schema and Realm-level permissions cannot be modified by anyone but admin users.
let realmPermissions = realm.objects(RealmPermission.self).first!
let everyonePermission = realmPermissions.permissions.findOrCreate(forRoleNamed: "everyone")
everyonePermission.canModifySchema = false
// `canSetPermissions` must be disabled last, as it would otherwise prevent other permission changes
// from taking effect.
everyonePermission.canSetPermissions = false
```

## Conclusion {#conclusion}

We've seen through this tutorial how easy it is to modify our earlier partial sync demo app to provide a level of security that prevents a user from accessing projects that don't belong to them.

We've discussed the default permissions created by Realm Cloud, and how to tighten these permissions before deploying your application.

We've covered changing permissions at a granularity of individual object instances, for an entire class, or the entire Realm.

This tutorial has focused on the high level concepts related to the new permission system. To see the entire source for the application, [clone the demo app repository from GitHub](https://github.com/realm/my-first-realm-app).

{% hint style="info" %}
The demo app is located under `ios/ObjectPermissions`. Don't forget to update the `Constants.swift` file with your Realm Cloud instance URL before running the app.
{% endhint %}

