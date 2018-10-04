# Fine-grained permissions

## Overview

In addition to [path-level permissions](path-level-permissions.md), you can control user access to individual objects through fine-grained permissions. This feature is in effect whenever the user is connecting to a Realm file using [Query-based synchronization](../syncing-data.md#using-query-based-synchronization).

The fine-grained permission system is based on [role-based access control lists](https://en.wikipedia.org/wiki/Role-based_access_control) and as such, permissions are assigned to [roles](fine-grained-permissions.md#roles) and not users directly. This means that a users current privileges is the sum of all roles the user is given.

The permission system recognizes three levels of permissions: [Realm](fine-grained-permissions.md#realm-level-permissions), [Class](fine-grained-permissions.md#class-level-permissions), and [Object-level](fine-grained-permissions.md#object-level-permissions). The hierarchy works like this: If a user does not have higher-level `Read` access, they cannot see anything on the lower levels. For example, if a user lacks `Read` access at the Realm-level, they cannot see any data in the Realm, even if they have `Read` at the class or object-level. However, just because they have higher-level `Read` access does not mean they can see _everything_ on the lower level -- just that they can see anything at all. In this way, the app developer can decide for themselves what granularity they want for permissions in their data model.

![Overview of fine-grained permissions](../../.gitbook/assets/fine-grained-permissions.png)

When first starting, any user who can connect to a Realm file can make any change to the data. This is designed to allow you to get up and running quickly, syncing data, and iterating on your data models. Once you are ready to start implementing access control, an admin can then define access control lists inside the Realm file, either on individual objects or whole classes.

Realm is an offline-first database, but permission checks are performed by the server. When a user makes a change to their local database, it will eventually be uploaded to the server, but if the server determines that the user tried to make changes that they were not allowed to make, the server will refuse to integrate it and instruct the client to revert the change. This all happens transparently from the perspective of the app.

{% hint style="info" %}
A reversal of an illegal change looks like any other change coming from the server.
{% endhint %}

To minimize the risk of a user unintentionally making illegal changes while offline, the permission metadata is replicated to each client for offline access, and can be [queried in the app for UI purposes](fine-grained-permissions.md#user-privileges). No permission checks are done automatically by the client -- the app has to manually query the permission metadata.

## Roles

The fine-grained permission model is based on roles. Roles are granted permissions, not users, but users, in turn, are assigned roles. This way, a [users privileges](fine-grained-permissions.md#user-privileges) are the sum of all roles they are part of.

A user can be a member of as many roles as needed. All users are automatically added to the special role called `everyone` by the server when they first connect.

A unique role is also automatically created for every user in the system. This role is named `__User:<syncIdentity>`.  The user is also automatically a member of this role. 

{% hint style="warning" %}
In a new Realm file, the `everyone` role has all permissions enabled.
{% endhint %}

When you as a developer are ready to integrate permissions in your app, you would usually define a new `administrator` role which has yourself as a member, and then reduce the privileges of the `everyone` role. Note that an `administrator` role is different than an `admin` user on the Realm ObjectServer. 

When assigning a role to a _User_, the user is added as a member of the _Role_ object instead of attaching the role to the user. Since the Role object is just a normal Realm object, it can be found, queried and manipulated the same way as other objects:

{% tabs %}
{% tab title="Swift" %}
```swift
// List all roles
let roles = realm.objects(PermissionRole.self);
​
// You can query roles in order to find a specific one
let role = realm.objects(PermissionRole.self).filter("name = 'my-role'").first;
  
// Making changes to a Role requires a write transaction
let user = getUserId();
try! realm.write {
  role.users.append(user)
}

// So does creating a new role
try! realm.write {
  let newRole = realm.create(PermissionRole.self, value: ["my-new-role"])
} 
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
// Comming soon
```
{% endtab %}

{% tab title="Java" %}
```java
// List all roles
RealmResults<Role> roles = realm.getRoles();

// You can query roles in order to find a specific one
Role role = realm.getRoles().where()
  .equalTo("name", "my-role")
  .findFirst();
  
// Making changes to a Role requires a write transaction
String user = getUserId();
realm.executeTransaction((Realm r) -> {
  role.addMember(user);
}); 

// So does creating a new role
realm.executeTransaction((Realm r) -> {
  r.insert(new Role("my-new-role"));
});
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
// List all roles
let roles = realm.objects(Realm.Permissions.Role);

// You can query roles in order to find a specific one
let role = realm
  .objects(Realm.Permissions.Role)
  .filtered(`name = 'my-role'`)[0];
  
// Making changes to a Role requires a write transaction
let user = getUser();
realm.write(() => {
  role.members.push(user);
})

// So does creating a new role
realm.write(() => {
  realm.create(Realm.Permissions.Role, { name: "my-new-role" });
});
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// Comming soon
```
{% endtab %}
{% endtabs %}

## Granting permissions

Once you have a [Role](fine-grained-permissions.md#roles), it can be granted permissions. This is done by creating a `Permission` object containing that role:

{% tabs %}
{% tab title="Swift" %}
```swift
// Create Permission object that grants read and update privileges to a Role
try! realm.write {
  let permission = realm.create(Permission.self);
  permission.role = getRole();
  permission.canRead = true;
  permission.canUpdate = true;
}  
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
// Comming soon
```
{% endtab %}

{% tab title="Java" %}
```java
// Create Permission object that grants read and update privileges to a Role
Role role = getRole();
Permission permission = new Permission.Builder(role)
  .canRead(true)
  .canUpdate(true)
  .build();  
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
// Create Permission object that grants read and update privileges to a Role
let permission = realm.create(Realm.Permissions.Permission, {
  role: getRole(),
  canRead: true,
  canUpdate: true,
});  
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// Comming soon
```
{% endtab %}
{% endtabs %}

By itself, the _Permission_ object does nothing and the privileges described in the object are not enforced until the object has been added to an appropriate permission list at either the [Realm-level](fine-grained-permissions.md#realm-level-permissions), the [Class-level](fine-grained-permissions.md#class-level-permissions) or the [Object-level](fine-grained-permissions.md#class-level-permissions). See the respective subsections for details on how to do this.

{% hint style="info" %}
A single _Permission_ object can be used in multiple places at the same time. This can be useful if you easily want to update privileges across multiple objects at the same time.
{% endhint %}

The following privileges can be set when creating or modifying the _Permission_ object:

* `canCreate`
* `canRead`
* `canUpdate`
* `canDelete`
* `canSetPermissions`
* `canQuery`
* `canModifySchema`

These privileges have different semantics depending on the level they are applied. See the below sections for the meaning of each privilege at the given level.

## Realm-level permissions

Realm-level permissions apply globally to the Realm file and are modified through a special singleton Realm object that is accessed the following way:

{% tabs %}
{% tab title="Swift" %}
```swift
// List all Realm-level permissions
let realmPermissions = realm.permissions;

// Find Realm level permissions for a given role
let rolePermissions = realmPermissions.filter("role.name = 'my-role'").first;
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
// Comming soon
```
{% endtab %}

{% tab title="Java" %}
```java
// Get the wrapper object for all Realm-level permissions
RealmPermissions realmPermissions = realm.getPermissions()

// List all Realm-level permissions
RealmList<Permission> allPermissions = realmPermissions.getPermissions()

// Find permissions for a given role
Permission rolePermissions = realmPermissions.getPermissions()
  .equalTo("role.name", "my-role")
  .findFirst()
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
// Get the global Realm-level permissions object
let realmPermissions = realm.permissions();

// Find permissions for a given role
let rolePermissions = realmPermissions.permissions
  .filtered(`role.name = 'my-role'`)[0];
 
// List the Realm-level permissions for all roles   
let allPermissions = realmPermissions.permissions;
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// Comming soon
```
{% endtab %}
{% endtabs %}

Adding Realm-level permissions for a role is done by adding a new [Permission object](fine-grained-permissions.md#granting-permissions) to the list of Realm-level permissions. This requires a write transaction:

{% tabs %}
{% tab title="Swift" %}
```swift
// Adding new permissions must be done within a write transaction
try! realm.write {

  // Grant read-only access at the Realm-level, which means
  // that users with this role can read all objects in the Realm
  // unless restricted by Class or Object level permissions.  
  let permissions = realm.permissions.findOrCreate(forRoleNamed: "my-role");
  permissions.canRead = true;
  permissions.canQuery = true;
}
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
// FIXME - Must match JS
// Example creating a new role with only read access to the Realm
 
// Permissions must be modified inside a write transaction
[realm transactionWithBlock:^{
    // Create the role
    RLMPermissionRole *readOnlyRole = [RLMPermissionRole createInRealm:realm withValue:@[@"read-only"]];

    // Add the user to the role
    RLMPermissionUser *user = getUser();
    [readOnlyRole.users addObject:user];

    // Create a new permission object for the role and add it to the Realm permissions
    RLMPermission *permission = [RLMPermission permissionForRoleNamed:readOnlyRole.name onRealm:realm];
    permission.canRead = YES;
    permission.canQuery = YES;
}];
```
{% endtab %}

{% tab title="Java" %}
```java
// Adding new permissions must be done within a write transaction
realm.executeTransaction((Realm r) -> {
  RealmPermissions realmPermissions = realm.getPermissions()

  // Grant read-only access at the Realm-level, which means
  // that users with this role can read all objects in the Realm
  // unless restricted by Class or Object level permissions.
  Permission permissions = realmPermissions.findOrCreate("my-role")
  permissions.setCanRead(true)
  permissions.setCanQuery(true)
 });
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
// Adding new permissions must be done within a write transaction
realm.write(() => {
  let realmPermissions = realm.permissions().permissions;
  
  // Grant read-only access at the Realm-level, which means
  // that users with this role can read all objects in the Realm
  // unless restricted by Class or Object level permissions.
  let role = realm.objects(Realm.Permissions.Role)
    .filtered(`name = 'my-role'`)[0];
  let permission = realm.create(Realm.Permissions.Permission, {
    role: role,
    canRead: true,
    canQuery: true,
  });  
  
  // Add it to the list of permissions for it to take affect.
  realmPermissions.push(permission);
});
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// FIXME - Must match JS
// Example creating a new role with only read access to the Realm
 
// Permissions must be modified inside a write transaction
realm.Write(() =>
{
    // Find or create the role
    var readOnlyRole = PermissionRole.Get(realm, "read-only");
     
    // Add the user to the role
    var user = User.Current;
    readOnlyRole.Users.Add(user);
     
    // Create a new permission object for the role and add it to the Realm
    // permissions
    var permission = Permission.Get(realm, readOnlyRole);
    permission.CanRead = true;
    permission.CanQuery = true;
});
```
{% endtab %}
{% endtabs %}

Modifying the Realm-level permissions for an existing role is done by finding the permission object for that role and modifying it. This requires a write transaction:

{% tabs %}
{% tab title="Swift" %}
```swift
// Modifying permissions must be done within a write transaction
try! realm.write {

  // Find permissions for the specific role
  let permissions = realm.permissions.findOrCreate(forRoleNamed: "my-role")

  // Prevent `my-role` users from modifying any objects in the Realm.
  permission.setCanUpdate(false)
  permission.setCanDelete(false)   
}
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
// Comming soon
```
{% endtab %}

{% tab title="Java" %}
```java
// Modifying permissions must be done within a write transaction
realm.executeTransaction((Realm r) -> {
  RealmPermissions realmPermissions = realm.getPermissions();
  
  // Find permissions for the specific role
  Permission permission = realmPermissions.findOrCreate("my-role");
  
  // Prevent `my-role` users from modifying any objects in the Realm.
  permission.setCanUpdate(false);
  permission.setCanDelete(false);    
});
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
// Modifying permissions must be done within a write transaction
realm.write(() => {
  let realmPermissions = realm.permissions().permissions;
  
   // Find permissions for the specfic role
  let permission = realmPermissions.filtered('role.name', 'my-role')[0];
  
  // Prevent `my-role` users from modifying any objects in the Realm.
  permission.canUpdate = false;
  permission.canDelete = false;
});
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// Comming soon
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Normal users can only grant other people access up to the level they themselves have, and only if they have the `SetPermissions` privilege. Realm Object Server `admin` users can see and edit everything.
{% endhint %}

The following table describes the effect of enabling or disabling privileges at the Realm level. If a privilege is marked with _N/A_ it means that the privilege has no meaning at this level beyond being required to enable setting it at Class or Object level.

| Type | ENABLED | DISABLED |
| :--- | :--- | :--- |
| canCreate | _N/A_ | _N/A_ |
| canRead | Role can see the Realm file itself, i.e. being able to meaningfully connect to it. Being able to see individual classes are covered by [Class-level permissions](fine-grained-permissions.md#class-level-permissions). **** | Role cannot see anything in the Realm file. An user that is offline will still create the file and schema locally on the device, but as soon as the device connects to the server, it will revert all changes \(leaving an empty Realm\). This will most likely crash the app. |
| canUpdate | Role can make changes to objects in the Realm file. This does not include schema changes or changes to permissions which are covered by `canModifySchema` and`canSetPermissions`.  | Role cannot change anything in the Realm file. It is read-only. Note that the user can write changes to the Realm file while offline, but any change will be reverted by the server once the device is online again.  |
| canDelete | _N/A_ | _N/A_ |
| canQuery | _N/A_ | _N/A_ |
| canSetPermissions | Role can set Realm-level permissions. A user with the`SetPermissions` privilege can never give other users higher privileges than they themselves have. | Role cannot set or change Realm-level permissions. |
| canModifySchema | Role can add classes to the schema, but not properties which are covered by [Class-level permissions](fine-grained-permissions.md#class-level-permissions). | Role cannot modify the schema in the entire Realm. |

## Class-level permissions

Class-level permissions are permissions related to all objects of a given type. 

They can be used reduce the scope of a Realm-level privilege, but cannot be used to broaden a privilege granted at the Realm-level. E.g. if `canRead` is set at the Realm-level, it is possible to set `canRead` to false at the Class-level, which will prevent the Role from seeing just this single class. However, if `canRead` was not set set the Realm-level, the value of `canRead` at the Class-level will be ignored and the class will not be readable.

Just like Realm-level permissions,  Class-level permissions are exposed as Realm objects that can be accessed the following way:

{% tabs %}
{% tab title="Swift" %}
```swift
// List the Class-level permissions for the given model class
let classPermissions = realm.permissions(forType: Person.self);
  
  // Find Class-level permissions for a given role
let rolePermissions = classPermissions.filter("role.name = 'my-role'").first
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
// Comming soon
```
{% endtab %}

{% tab title="Java" %}
```java
// Get the Class-level permissions object for the Person class
ClassPermissions classPermissions = realm.getPermissions(Person.class);
  
// List the Class-level permissions for all roles
RealmList<Permission> permissions = classPermissions.getPermissions();

// Find Class-level permissions for a given role
Permission rolePermissions = classPermissions.getPermissions().where()
  .equalTo("role.name", "my-role")
  .findFirst();
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
// Get the Class-level permissions object for the Person class
let classPermissions = realm.permissions('Person');

// List the Class-level permissions for all roles
let permissions = classPermissions.permissions;

// Find Class-level permissions for a given role
let rolePermissions = classPermissions.permissions
  .filtered(`role.name = 'my-role'`)[0];
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// Comming soon
```
{% endtab %}
{% endtabs %}

Adding Class-level permissions for a role is done by adding a new [Permission object](fine-grained-permissions.md#granting-permissions) to the list of permissions. This requires a write transaction:

{% tabs %}
{% tab title="Swift" %}
```swift
// Adding new permissions must be done within a write transaction
try! realm.write {

  // Remove read-access for the Person class for users with the `my-role` role.
  let permission = realm
    .permissions(forType: Person.self)
    .findOrCreate(forRoleNamed: "my-role")
  permission.canRead = false
});
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
// Comming soon
```
{% endtab %}

{% tab title="Java" %}
```java
// Adding new permissions must be done within a write transaction
realm.executeTransaction((Realm r) -> {
  ClassPermissions realmPermissions = realm.getPermissions(Person.class);
  
  // Remove read-access for the Person class for users with the `my-role` role.
  Role role = realm.where(Role.class).equalTo("name", "my-role").findFirst();
  Permission p = new Permission.Builder(role)
    .canRead(false)
    .build();
      
  // Add it to the list of permissions for it to take affect.
  classPermissions.getPermissions().add(permission);  
});
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
// Adding new permissions must be done within a write transaction
realm.write(() => {
  let realmPermissions = realm.privileges('Person').permissions;
  
  // Remove read-access for the Person class for users with `my-role` role.
  let role = realm.objects(Realm.Permissions.Role)
    .filtered(`name = 'my-role'`)[0];
    
  let permission = realm.create(Realm.Permissions.Permission, {
    role: role,
    canRead: false,
  });  
  
  // Add it to the list of permissions for it to take affect.
  classPermissions.push(permission);
});
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// Comming soon
```
{% endtab %}
{% endtabs %}

Modifying the Class-level permissions for an existing role is done by finding the permission object for that role and modify it. This requires a write transaction:

{% tabs %}
{% tab title="Swift" %}
```swift
// Modifying permissions must be done within a write transaction
try! realm.write {

  // Find permissions for the role and change it
  let permission = realm
    .permissions(forType: Person.self)
    .findOrCreate(forRoleNamed: "my-role")

    // Prevent `my-role` users from modifying any Person objects.
  permission.setCanUpdate(false)
  permission.setCanDelete(false)
});
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
// Comming soon
```
{% endtab %}

{% tab title="Java" %}
```java
// Modifying permissions must be done within a write transaction
realm.executeTransaction((Realm r) -> {
  ClassPermissions classPermissions = realm.getPermissions(Person.class);
  
  // Find permissions for the role and change it
  Permission permission = classPermissions.getPermissions().where()
    .equalTo("role.name", "my-role")
    .findFirst();
  
  // Prevent `my-role` users from modifying any Person objects.
  permission.setCanUpdate(false);
  permission.setCanDelete(false);    
});
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
// Modifying permissions must be done within a write transaction
realm.write(() => {
  let classPermissions = realm.privileges('Person').permissions;
  
   // Find permissions for a specfic role
  let permission = classPermissions.filtered('role.name', 'my-role')[0];
  
  // Prevent `my-role` users from modifying any objects in the Realm.
  permission.canUpdate = false;
  permission.canDelete = false;
});
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// Comming soon
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Normal users can only grant other people access up to the level they themselves have, and only if they have the `SetPermissions` privilege. Admin users can see and edit everything.
{% endhint %}

The following table describes the effect of enabling or disabling a given privilege at the Class level. If a privilege is marked with _N/A_ it means that the privilege has no meaning at this level beyond being required to enable setting it at the Object level.

<table>
  <thead>
    <tr>
      <th style="text-align:left">Type</th>
      <th style="text-align:left">ENABLED</th>
      <th style="text-align:left">DISABLED</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">canCreate</td>
      <td style="text-align:left">
        <p>Role can create objects of this type.</p>
        <p>If a user has the <code>canCreate</code> privilege, but not <code>canUpdate,</code>they
          are still able to modify newly-created objects inside the same transaction
          that object was created in.</p>
      </td>
      <td style="text-align:left">Role cannot create objects of this type.</td>
    </tr>
    <tr>
      <td style="text-align:left">canRead</td>
      <td style="text-align:left">Role can see objects of this type.</td>
      <td style="text-align:left">Role cannot see any objects of this type. It is still allowed to query
        them, but the query result will be empty</td>
    </tr>
    <tr>
      <td style="text-align:left">canUpdate</td>
      <td style="text-align:left">Role can change properties in objects of this type.</td>
      <td style="text-align:left">Role cannot make any change to properties in objects of this type.</td>
    </tr>
    <tr>
      <td style="text-align:left">canDelete</td>
      <td style="text-align:left"><em>N/A</em>
      </td>
      <td style="text-align:left"><em>N/A</em>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">canQuery</td>
      <td style="text-align:left">Role is allowed to create a server side <a href="../syncing-data.md#subscribing-to-data">Subscription</a> for
        this type.</td>
      <td style="text-align:left">Role is not allowed to create a server side Subscription for this type. <em>Note: Local queries will always work.</em>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">canSetPermissions</td>
      <td style="text-align:left">Role can set Class-level permissions for the class.</td>
      <td style="text-align:left">Role cannot set class-level permissions for the class.</td>
    </tr>
    <tr>
      <td style="text-align:left">canModifySchema</td>
      <td style="text-align:left">Role can add properties to this class. Deleting and renaming fields is
        currently not supported by the Realm Object Server.</td>
      <td style="text-align:left">Role cannot modify the schema for this class.</td>
    </tr>
  </tbody>
</table>## Object-level permissions

Object-level permissions are permissions related to a single Realm object. 

They can be used to reduce the scope of Class-level privileges, but cannot be used to broaden a privilege granted at the Class-level. E.g. if `canRead` is enabled at the Class-level, it is possible to disable `canRead` at the Object-level which will prevent the Role from seeing just this single object. However, if `canRead` was disabled at the Class-level, no matter the value of `canRead` at the Object-level the object will not be readable.

In order to add permissions to your objects, you must add a special Access Control List \(ACL\) property. If your objects do not have the ACL property, the objects will be fully accessible by anyone \(provided that they are discoverable -- see the `canQuery/canRead` permission at the Class level\).

The ACL property is added the following way:

{% tabs %}
{% tab title="Swift" %}
```swift
// The ACL property is a `List<Permission>` field with a user-defined name
class Person: Object {
    @objc dynamic var name = ""
    let permissions = List<Permission>()
}
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
// The ACL property is a `RLMArray<RLMPermission *>` field 
// with a user-defined name
@interface Person : RLMObject
@property NSString *name;
@property RLMArray<RLMPermission *><RLMPermission> *permissions;
@end
```
{% endtab %}

{% tab title="Java" %}
```java
// The ACL property is a `RealmList<Permission>` field 
// with a user-defined name
public class Person extends RealmObject {
  public String name;
  public RealmList<Permission> permissions = new RealmList<>();
}
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
// The ACL property is a list-of-permission property 
// with a user-defined name
const PersonSchema = {
    name: 'Person',
    properties: {
        name: 'string',
        permissions: '__Permission[]'
    }
};
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// The ACL property is an `IList<Permission>` property with a user-defined name
public class Person : RealmObject
{
    // Other properties ...

    public IList<Permission> Permissions { get; }
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
If there is an ACL property, but no permissions have been added for any role, then nobody except Realm Object Server admin users will have access to the object. This  includes the user creating the object.
{% endhint %}

Adding Object-level permissions for a role is done adding a new [Permission object](fine-grained-permissions.md#granting-permissions) to the list of permissions. This requires a write transaction:

{% tabs %}
{% tab title="Swift" %}
```swift
// Adding new permissions must be done within a write transaction
try! realm.write {

  // Grant users with the `shared-objects` role access to read and modify
  // this object
  let person = getPerson();
  let permissions = person.permissions.findOrCreate(forRoleNamed: "shared-objects);
  permissions.canRead = true
  permissions.canUpdate = true
  permissions.canDelete = true
}
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
// Comming soon 
```
{% endtab %}

{% tab title="Java" %}
```java
// Adding new permissions must be done within a write transaction
realm.executeTransaction((Realm r) -> {
  Person p = realm.where(Person.class).equalto("id", getId()).findFirst()
  
  // Grant users with the `shared-objects` role access to read and modify
  // this object
  Role role = realm.where(Role.class)
    .equalTo("name", "shared-objects")
    .findFirst();
  Permission p = new Permission.Builder(role)
    .canRead(true)
    .canUpdate(true)
    .canDelete(true)
    .build();
      
  // Add it to the list of permissions associated with the object
  // for it to take affect.
  p.permissions.add(permission);  
});
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
// Adding new permissions must be done within a write transaction
realm.write(() => {
  let person = realm.objects('Person').filtered(`id = "my-id"`)[0];
  
  // Grant users with the `shared-objects` role access to read and modify
  // this object
  let role = realm.objects(Realm.Permissions.Role)
    .filtered(`id = "shared-objects"`)[0];
  let permission = realm.create(Realm.Permissions.Permission, {
    role: role,
    canRead: true,
    canUpdate: true,
    canDelete: true,
  });  
  
  // Add it to the list of permissions for it to take affect.
  person.permissions.push(permission);
});
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// Comming soon 
```
{% endtab %}
{% endtabs %}

Modifying the Object-level permissions for an existing role is done by finding the permission object for that role and modifying it. This requires a write transaction:

{% tabs %}
{% tab title="Swift" %}
```swift
// Modifying permissions must be done within a write transaction
try! realm.write {
  let person = getPerson()

  // Prevent `shared-objects` users from modifying this object.  
  let permissions = person.permissions.findOrCreate(forRoleNamed: "shared-objects)
  permissions.canUpdate = false
  permissions.canDelete = false
}
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
// Comming soon
```
{% endtab %}

{% tab title="Java" %}
```java
// Modifying permissions must be done within a write transaction
realm.executeTransaction((Realm r) -> {
  Person p = realm.where(Person.class).equalTo("id", getId()).findFirst();
  
  // Find permissions for a specific role and change them
  Permission permission = p.permissions.where()
    .equalTo("role.name", "shared-objects")
    .findFirst();
  
  // Prevent `shared-objects` users from modifying this object.
  permission.setCanUpdate(false);
  permission.setCanDelete(false);    
});
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
// Modifying permissions must be done within a write transaction
realm.write(() => {
  let person = realm.objects('Person')
    .filtered(`id = "${getId()}"`)[0];
  
  // Find permissions for a specific role and change them
  let permission = person.permissions('role.name', 'shared-objects')[0];
  
  // Prevent `shared-objects` users from modifying this object.
  permission.canUpdate = false;
  permission.canDelete = false;
});
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// Comming soon
```
{% endtab %}
{% endtabs %}

Any object that is reachable through links or collections from an object to which a user has Read access will also be readable by that user, even if they explicitly do not have Read access to the reachable object. This is due to a current limitation in the way object graphs are represented in Realm.

The following table describes the effect of enabling or disabling a given privilege at the Object level:

| Type | ENABLED | DISABLED |
| :--- | :--- | :--- |
| canCreate | _N/A_ | _N/A_ |
| canRead | Role can see and read this object. This include any referenced objects. | Role cannot read or see this object. |
| canUpdate | Role can update properties on this object. The ACL property is a special case handled by `canSetPermissions`. | Role is not allowed to change the value of any properties on the object. |
| canDelete | Role can delete the object. | Role is not allowed to delete the object. |
| canQuery | _N/A_ | _N/A_ |
| canSetPermissions | Role can modify Permission objects referenced by the custom ACL property on the object. | Role is not allowed to modify the custom ACL property on the object. |
| canModifySchema | _N/A_ | _N/A_ |

## User Privileges

Because a `User` can be part of multiple roles with many different permissions, it is often useful to be able to determine exactly what the current user can do with an object \(or class, or the Realm file\).

This can be achieved in the following way:

{% tabs %}
{% tab title="Swift" %}
```swift
// Realm privileges
let privileges = realm.getPrivileges()
 
// Class privileges for `Person`
let privileges = realm.getPrivileges(Person.self)
 
// Object privileges
let person = getPerson()
let privileges = realm.getPrivileges(person)
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
// Realm privileges
struct RLMRealmPrivileges privileges = [realm privilegesForRealm];
 
// Class privileges for `Person`
struct RLMClassPrivileges privileges = [realm privilegesForClass:Person.class];
 
// Object privileges
Person *person = getPerson();
struct RLMObjectPrivileges privileges = [realm privilegesForObject:person];
```
{% endtab %}

{% tab title="Java" %}
```java
// Realm privileges
RealmPrivileges privileges = realm.getPrivileges();

// Class privileges for `Person`
ClassPrivileges privileges = realm.getPrivileges(Person.class);

// Object privileges
Person person = getObject();
ObjectPrivileges privileges = realm.getPrivileges(person);
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
// Realm privileges
let privileges = realm.privileges();

// Class privileges for `Person`
let classPrivileges = realm.privileges('Person');

// Object privileges
let person = getPerson();
let objectPrivileges = realm.getPrivileges(person);
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// Realm privileges
var privileges = realm.GetPrivileges();
 
// Class privileges for Person
var privileges = realm.GetPrivileges<Person>();

// Class privileges for Person using the string API
var privileges = realm.GetPrivileges("Person");
 
// Object privileges
var person = realm.Find<Person>(someId);
var privileges = realm.GetPrivileges(person);
```
{% endtab %}
{% endtabs %}

This can e.g. be used to toggle an Edit button if a user only have read access to an object:

{% tabs %}
{% tab title="Swift" %}
```swift
let person = getPerson()
let privileges = realm.getPrivileges(person)

if privileges.contains(.update) {
    showEditButton()
}
else {
    hideEditButton()
}
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
Person *person = getPerson();
struct RLMObjectPrivileges privileges = [realm privilegesForObject:person];

if (privileges.update) {
    showEditButton()
}
else {
    hideEditButton()
}
```
{% endtab %}

{% tab title="Java" %}
```java
Person person = getObject();
ObjectPrivileges privileges = realm.getPrivileges(person);

if (privileges.canUpdate()) {
  showEditButton();
} else {
  hideEditButton();
}
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
let person = getPerson();
let objectPrivileges = realm.getPrivileges(person);

if (privileges.canUpdate) {
  showEditButton();
} else {
  hideEditButton();
}
```
{% endtab %}

{% tab title=".Net" %}
```csharp
var person = realm.Find<Person>(someId)
var privileges = realm.GetPrivileges(person);

if (privileges.HasFlag(ObjectPrivileges.Update))
{
    ShowEditButton();
}
else
{
    HideEditButton();
}
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
If a user does not have `canRead` access to the classes used by the permission system, querying about privileges will always return _false_ , even if the user might have access. See the [next section](fine-grained-permissions.md#restricting-access-to-permission-metadata) for more details.
{% endhint %}

## Restricting access to Permission metadata

Fine-grained permissions are implemented using Realm classes and objects. This means that the permission system itself is subject to the same permission rules as normal model classes. 

As a consequence it is possible to disallow users access to the information in the permission system by removing the `canRead` privilege for those classes. This is generally not recommended as it prevent users from knowing the full extend of their [privileges](fine-grained-permissions.md#user-privileges). 

The classes used for implementing the fine-grained permission system are:

{% tabs %}
{% tab title="Swift" %}
```swift
// Comming soon
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
// Comming soon
```
{% endtab %}

{% tab title="Java" %}
```java
io.realm.sync.permissions.PermissionUser
io.realm.sync.permissions.Permission
io.realm.sync.permissions.RealmPermissions
io.realm.sync.permissions.ClassPermissions
io.realm.sync.permissions.Role

See the API docs here: 
​https://realm.io/docs/java/latest/api/io/realm/sync/permissions/package-summary.html 
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
// Comming soon
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// Comming soon
```
{% endtab %}
{% endtabs %}

Specifically it means that:

* A user cannot modify any Realm-level permissions unless they have the `setPermissions` privilege on the Class-level permission object for the `__Realm` class.
*  A user cannot modify any Class-level permissions unless they have the `setPermissions` privilege on the Class-level permission object for the `__Class` class.

This can be verified the following way:

{% tabs %}
{% tab title="Swift" %}
```swift
// Comming soon
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
// Comming soon
```
{% endtab %}

{% tab title="Java" %}
```java
// Check access to Realm-level permissions
RealmPrivileges realmPrivs = realm.getPrivileges(RealmPermissions.class);
realmPrivs.canRead(); // Can see Realm-level permissions
realmPrivs.canSetPermissions(); // Can modify Realm-level permissions

// Check access to Class-level permissions
ClassPrivileges classPrivs = realm.getPrivileges(ClassPermissions.class);
classPrivs.canRead(); // Can see Class-level permissions
classPrivs.canSetPermissions(); // Can modify Class-level permissions
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
// Check access to Realm-level permissions
let realmPrivileges = realm.privileges(Realm.Permissions.Realm);
realmPrivileges.canRead; // Can see Realm-level permissions
realmPrivileges.canSetPermissions; // Can modify Realm-level permissions

// Check access to Class-level permissions
let classPrivileges = realm.privileges(Realm.Permissions.Class);
classPrivileges.canRead; // Can see Class-level permissions
classPrivileges.canSetPermissions; // Can modify Class-level permissions
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// Comming soon
```
{% endtab %}
{% endtabs %}

## Realm Studio

The fine-grained permission data can be seen and edited from [Realm Studio](../../realm-studio/). In order to see it, you need to enable "Show system classes" Under the "View" menu item. 

![Realm Studio showing fine-grained permission classes](../../.gitbook/assets/image%20%281%29.png)

They are stored in the tables named `__Realm` , `__Class`, `__Role`, `__User` and `__Permission`.

Realm-level permissions are defined in a special object with `id = 0` in the `__Realm` table.

Class-level permissions are stored in the `__Class` table. Each Realm class has an entry in that table. 

## Examples

_Coming soon_

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3)

