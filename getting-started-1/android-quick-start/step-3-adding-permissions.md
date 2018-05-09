# Step 3 - Adding Permissions

In the previous tutorial we used partial sync to synchronize only projects owned by the current user. However, nothing prevented a malicious user from seeing another user's projects by subscribing with a query that matches a broader set of `Project` instances.

To address this issue we will instead use Realm's permission system to limit access to each `Project` instance so only the user that created it can read, modify, or delete it.

## Quick Start  {#quick-start}

{% hint style="success" %}
Want to get started right away with the complete source code? [Clone the demo app repository from GitHub](https://github.com/realm/my-first-realm-app), then follow the instructions in `android/objectPermission` to get started. Don't forget to update the `Constants.java` file with your Realm Cloud instance URL before running the app.
{% endhint %}

## Updating the ToDo application {#updating-the-todo-application}

### Control access to `Project` instances {#control-access-to-project-instances}

In the previous tutorial we used to perform partial sync by filtering the projects by their owners. However nothing prevents a malicious user to modify someone else's project by registering a query for all `Project`.

To alleviate this issue, we can choose to restrict the access to the `Project` instance so only the user who created it, can read and modify it. Thanks to the new permission system, we can implement this restriction by adding an ACL to the `Project` model.

```java
public class Project extends RealmObject {
// ...
    private RealmList<Permission> permissions;
}
```

The `Permission` class defines the privileges we want to grant and to whom \(which _Role_\). In our scenario we want to create a user specific Role, then grant all permissions to it. This has the consequence to prevent all other users from seeing and modifying our `Project` instance.

By default, every logged-in user has a private role created for them. This role can be accessed at `PermissionUser.getPrivateRole()`. We'll use this role when creating new projects.

### Associate a `Permission` object with the created `Project`  {#associate-a-permission-object-with-the-created-project}

As we create a new `Project` we now associate with it a new `Permission` instance to protect it.

```java
String name = taskText.getText().toString();
Project project = realm.createObject(Project.class, UUID.randomUUID().toString());
project.setName(name);
project.setTimestamp(new Date());

// Create a restrictive permission to limit read/write access to the current user only
Role role = realm.where(PermissionUser.class)
        .equalTo("id", SyncUser.current().getIdentity())
        .findFirst()
        .getPrivateRole();
Permission permission = new Permission.Builder(role).allPrivileges().build();
project.getPermissions().add(permission);
```

{% hint style="info" %}
Note that we no longer need the `owner` attribute in `Project` model, it has been removed from the model.
{% endhint %}

## Lower the default permissions {#lower-the-default-permissions}

By default Realm Cloud creates a set of permissive default permissions and roles. This allows getting started on developing your application without first having to worry about permissions and roles. However, these permissive default permissions should be tightened before deploying your application.

By default, all users are added to an `everyone` role, and this role has access to all objects within the Realm. As part of lowering the default permissions, we will limit the access of this `everyone` role.

{% hint style="warning" %}
In this demo, we lower the privileges programmatically the first time a user logs in to our ToDo application. In practice, you'll want to configure these privileges using Realm Studio or a script prior to your users running the application.
{% endhint %}

There are two levels of permissions that we will lower: class-level permissions, and Realm-level permissions.

#### Class-level permissions {#class-level-permissions}

We use permissions to limit querying to only the `Project` type. This means that the only `Item` objects that can be synchronized are those associated with a `Project` that we have permissions to access. Additionally, we remove the ability for `everyone` to change the permissions of each of our model classes.

```java
// Lower "everyone" Role on Item & Project to restrict permission modifications
Permission itemPermission = realm.where(ClassPermissions.class).equalTo("name", "Item").findFirst().getPermissions().first();
Permission projectPermission = realm.where(ClassPermissions.class).equalTo("name", "Project").findFirst().getPermissions().first();
itemPermission.setCanQuery(false);
itemPermission.setCanSetPermissions(false);
projectPermission.setCanSetPermissions(false);
```

#### Realm-level permissions {#realm-level-permissions}

We use permissions to prevent the schema and permissions from being modified. Note that the order in which we perform these operations is significant, as removing the ability to modify permissions would cause subsequent permission changes to be rejected.

```java
// Lock the permission and schema
RealmPermissions permission = realm.where(RealmPermissions.class).equalTo("id", 0).findFirst();
Permission everyonePermission = permission.getPermissions().first();
everyonePermission.setCanModifySchema(false);
everyonePermission.setCanSetPermissions(false);
```

## Conclusion {#conclusion}

We've seen through this tutorial how easy it is to modify our earlier partial sync demo app to provide a level of security that prevents a user from accessing projects that don't belong to them.

We've discussed the default permissions created by Realm Cloud, and how to tighten these permissions before deploying your application.

We've covered changing permissions at a granularity of individual object instances, for an entire class, or the entire Realm.

This tutorial has focused on the high level concepts related to the new permission system. To see the entire source for the application, [Clone the demo app repository from GitHub](https://github.com/realm/my-first-realm-app).

