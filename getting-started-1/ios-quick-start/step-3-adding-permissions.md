---
description: >-
  In this tutorial, we're going to see how we can use Realm's permission system
  in order to limit which users can see our projects within our ToDo
  application.
---

# Step 3 - Adding Permissions

In the previous tutorial we used partial sync to synchronize only projects owned by the current user. However, nothing prevented a malicious user from seeing another user's projects by subscribing with a query that matches a broader set of `Project` instances.

To address this issue we will instead use Realm's permission system to limit access to each `Project` instance so only the user that created it can read, modify, or delete it.

## Quick Start {#quick-start}

This tutorial builds off the previous two steps. We recommend that you complete these first then continue with this. If not, you can follow the instructions below to clone the existing project:

{% hint style="info" %}
Want to get started right away with the complete source code? [Clone the demo app repository from GitHub](https://github.com/realm/my-first-realm-app), then follow the instructions in [`ios/ObjectPermissions/README.md`](https://github.com/realm/my-first-realm-app/blob/master/ios/ObjectPermissions/README.md) to get started. Don't forget to update the `Constants.swift` file with your Realm Cloud or self-hosted instance URL before running the app.
{% endhint %}

## Updating the ToDo application {#updating-the-todo-application}

### A user's role {#create-a-new-role-per-user}

Permissions are assigned to a collection of zero or more users named a _role_. Since we want a given `Project` to be accessible only to a single user, we need to use a separate role for each user. We can then grant the current user's role the appropriate permissions when we create a new `Project`. This will have the effect of preventing all other users from seeing or modifying that `Project` instance.

By default, every logged-in user has a private role created for them. This role can be accessed at `PermissionUser.role`. We'll use this role when creating new projects.

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

    let user = self.realm.object(ofType: PermissionUser.self, forPrimaryKey: SyncUser.current!.identity!)!
    let permission = project.permissions.findOrCreate(forRole: user.role!)
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
In this demo, we lower the privileges programmatically the first time a user logs in to our ToDo application. In practice, you'll want to configure these privileges using [Realm Studio](../../realm-studio/) or a script prior to your users running the application.
{% endhint %}

There are two levels of permissions that we will lower: class-level permissions, and Realm-level permissions.

#### Class-level permissions {#class-level-permissions}

We use permissions to limit querying to only the `Project` type. This means that the only `Item` objects that can be synchronized are those associated with a `Project` that we have permissions to access. Additionally, we remove the ability for `everyone` to change the permisisons of each of our model classes.

```swift
// Ensure that class-level permissions cannot be modified by anyone but admin users.
// The Project type can be queried, while Item cannot. This means that the only Item
// objects that will be synchronized are those associated with our Projects.
// Additionally, we prevent roles from being modified to avoid malicious users
// from gaining access to other user's projects by adding themselves as members
// of that user's private role.
let queryable = [Project.className(): true, Item.className(): false, PermissionRole.className(): true]
let updateable = [Project.className(): true, Item.className(): true, PermissionRole.className(): false]

for cls in [Project.self, Item.self, PermissionRole.self] {
    let everyonePermission = realm.permissions(forType: cls).findOrCreate(forRoleNamed: "everyone")
    everyonePermission.canQuery = queryable[cls.className()]!
    everyonePermission.canUpdate = updateable[cls.className()]!
    everyonePermission.canSetPermissions = false
}
```

#### Realm-level permissions {#realm-level-permissions}

We use permissions to prevent the schema and permissions from being modified. Note that the order in which we perform these operations is significant, as removing the ability to modify permissions would cause subsequent permission changes to be rejected.

```swift
// Ensure that the schema and Realm-level permissions cannot be modified by anyone but admin users.
let everyonePermission = realm.permissions.findOrCreate(forRoleNamed: "everyone")
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

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

