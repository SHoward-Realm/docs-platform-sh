---
description: Client applications can login multiple users for advanced scenarios.
---

# Working with multiple users

The Realm Platform allows an application to support multiple users at the same time.

For example, consider an email client app that supports connecting to multiple email accounts. Multiple users \(one for each email account\) can be authenticated at any given time. The Realm SDKs allows you to login multiple users then retrieve a specific user to open his/her Realm as needed.

{% tabs %}
{% tab title="Swift" %}
The `current` method will retrieve the **current user,** the last user who logged in and whose credentials have not expired. The method returns `nil` if no current user exists. An exception will be thrown if more than one logged-in user exists.

```swift
let user = SyncUser.current!
```

If there are multiple users logged in, you can get a dictionary of user objects corresponding to them with `all`.

```swift
let allUsers = SyncUser.all
```
{% endtab %}

{% tab title="Objective-C" %}
The `currentUser` method will retrieve the **current user,** the last user who logged in and whose credentials have not expired. The method returns `nil` if no current user exists. An exception will be thrown if more than one logged-in user exists.

```objectivec
RLMSyncUser *user = [RLMSyncUser currentUser];
```

If there are multiple users logged in, you can get a dictionary of user objects corresponding to them with `allUsers`.

```text
NSDictionary<NSString *, RLMSyncUser *> *allUsers = [RLMSyncUser allUsers];
```

If no users are logged in, the dictionary returned will be empty.
{% endtab %}

{% tab title="Java" %}
The `SyncUser.current()` method will retrieve the **current user,** the last user who logged in and whose credentials have not expired. The method returns `nil` if no current user exists. An exception will be thrown if more than one logged-in user exists.

```text
SyncUser user = SyncUser.current();
```

If there are multiple users logged in, you can get a map between their server identity and their corresponding `SyncUser` object.

```text
Map<String, SyncUser> users = SyncUser.all();
```
{% endtab %}

{% tab title="Javascript" %}
`Realm.Sync.User.current` can be used to obtain the currently logged in user. If no users have logged in or all have logged out, it will return `undefined`. If there are more than one logged in users, an error will be thrown.

```javascript
const user = Realm.Sync.User.current;
```

If there are likely to be multiple users logged in, you can get a collection of them by calling `Realm.Sync.User.all`. This will be empty if no users have logged in.

```javascript
let users = Realm.Sync.User.all;

for(const key in users) {
  const user = users[key];

  // do something with the user object.
}
```
{% endtab %}

{% tab title=".Net" %}
If you have not disabled user persistence, [User.Current](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.User.html#Realms_Sync_User_Current) can be used to obtain the currently logged in user. If no users have logged in or all have logged out, it will return `null`. If there are more than one logged in users, a [RealmException](https://realm.io/docs/dotnet/latest/api/reference/Realms.Exceptions.RealmException.html) will be thrown.

```csharp
var user = User.Current;
```

If there are likely to be multiple users logged in, you can get a collection of them by calling [User.AllLoggedIn](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.User.html#Realms_Sync_User_AllLoggedIn). This will be empty if no users have logged in.

```csharp
var users = User.AllLoggedIn;

foreach (var user in users)
{
    // do something with the user
}
```
{% endtab %}
{% endtabs %}



