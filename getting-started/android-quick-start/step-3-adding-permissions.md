# Step 3 - Adding Permissions

{% hint style="danger" %}
## This tutorial uses new APIs still in beta.

The API may change, and we encourage you to give feedback!
{% endhint %}

In the previous tutorial we used partial sync to synchronize only projects owned by the current user. However, nothing prevented a malicious user from seeing another user's projects by subscribing with a query that matches a broader set of `Project` instances.

To address this issue we will instead use Realm's permission system to limit access to each `Project` instance so only the user that created it can read, modify, or delete it.

## Quick Start  {#quick-start}

{% hint style="success" %}
Want to get started right away with the complete source code? [Clone the demo app repository from GitHub](https://github.com/realm/my-first-realm-app), then follow the instructions in `android/objectPermission` to get started. Don't forget to update the `Constants.java` file with your Realm Cloud instance URL before running the app.
{% endhint %}

## Setting up a new Realm with Realm Studio {#setting-up-a-new-realm-with-realm-studio}

Realm Studio is a powerful tool, one of the advanced features is to be able to create a Realm and assign permissions to it, without the need to do this programmatically.

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

{% hint style="info" %}
If you don't see the `__admin` Realm, go to the "View" menu and select "Show System Realms":

![](../../.gitbook/assets/image%20%283%29.png)
{% endhint %}

You can verify that new `ToDo-permissions` Realm is set up correctly by checking that its ownership is reported as `public R W`.

{% hint style="info" %}
Note: when creating the permission we left the field `user` empty \(`null`\) in the dialog box, this is a way to indicate that the permission will apply to all users \(`*`\)
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

### Add a specific Role per user {#add-a-specific-role-per-user}

After a successful login we use the current `SyncUser` to create a specific `Role` that we're going to use later.

```java
// Create a Role specific to this user
Role userRole = realm.createObject(Role.class, roleId);
// add current user to it
userRole.addMember(user.getIdentity());
```

### Associate a `Permission` object with the created `Project`  {#associate-a-permission-object-with-the-created-project}

As we create a new `Project` we now associate with it a new `Permission` instance to protect it.

```java
String name = taskText.getText().toString();
Project project = realm.createObject(Project.class, UUID.randomUUID().toString());
project.setName(name);
project.setTimestamp(new Date());

// Create a restrictive permission to limit read/write access to the current user only
Role role = realm.where(Role.class).equalTo("name", roleId).findFirst();
Permission permission = new Permission(role);
permission.setCanRead(true);
permission.setCanQuery(true);
permission.setCanCreate(true);
permission.setCanUpdate(true);
permission.setCanUpdate(true);
permission.setCanDelete(true);
permission.setCanSetPermissions(true);
permission.setCanModifySchema(true);

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

