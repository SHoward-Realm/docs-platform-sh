# Opening A Synced Realm

## Configuring The Realm To Be Opened

Before you can open a synced Realm, you must first provide a configuration for it. This process is similar to configuring local Realms, such as defining the local path on disk, but synced Realms have specific requirements and not all configuration options apply.

### The Default Synced Realm

The easiest way to get started with Realm Platform is to use the default synchronized Realm. With Realm Platform - Cloud each cloud instance will have a single default Realm. For Self-Hosted each server installation will include a single default Realm.

For most apps, you can include all of your application data within the default synchronized Realm. Realm Platform supports [query-based sync](syncing-data.md#partial-synchronization), so that you can control what data from the default synced Realm is synchronized to the client application.

In addition, to you application data, the server will also include other internal classes used in its management and the master list of authenticated users, represented by the `__User` class. The internal data is managed through [fine-grained access control](access-control/#permissions).

The purpose of the default synchronized Realm is to simplify getting started and allow your application data to link into the master list of authenticated users.

{% tabs %}
{% tab title="Swift" %}
The default synced Realm is provided via an automatic `SyncConfiguration`, which will use the current logged in user from `SyncUser.current` and the server URL used to authenticate.

For example, to log in, then asynchronously open the default synced Realm with the current user:

```swift
SyncUser.logIn(with: credentials, server: serverURL) { user, error in
    if let user = user {
        Realm.Configuration.defaultConfiguration = SyncConfiguration.automatic()
        Realm.asyncOpen() { realm in
            // ...
        }
    }
}
```

If you are working with multiple users, you can pass in the specific user as well:

```swift
SyncUser.logIn(with: credentials, server: serverURL) { user, error in
    if let user = user {
        Realm.Configuration.defaultConfiguration = SyncConfiguration.automatic(user: user)
        Realm.asyncOpen() { realm in
            // ...
        }
    }
}
```
{% endtab %}

{% tab title="Objective-C" %}
The default synced Realm is provided via an automatic `RLMSyncConfiguration`, which will use the current logged in user from `[RLMSyncUser currentUser]` and the server URL used to authenticate.

For example, to log in, then asynchronously open the default synced Realm with the current user:

```objectivec
[RLMSyncUser logInWithCredentials:credentials
                    authServerURL:serverURL
                     onCompletion:^(RLMSyncUser *user, NSError *error) {
    if (user) {
        RLMRealmConfiguration *config = [RLMSyncConfiguration automaticConfiguration];
        [RLMRealm asyncOpenWithConfiguration:config
                       callbackQueue:dispatch_get_main_queue()
                            callback:^(RLMRealm *realm, NSError *error) {
            if (realm) {
                // ...
            }
        }];
    }
}];
```

If you are working with multiple users, you can pass in the specific user as well:

```objectivec
[RLMSyncUser logInWithCredentials:credentials
                    authServerURL:serverURL
                     onCompletion:^(RLMSyncUser *user, NSError *error) {
    if (user) {
        RLMRealmConfiguration *config = [RLMSyncConfiguration automaticConfigurationForUser:user];
        [RLMRealm asyncOpenWithConfiguration:config
                       callbackQueue:dispatch_get_main_queue()
                            callback:^(RLMRealm *realm, NSError *error) {
            if (realm) {
                // ...
            }
        }];
    }
}];
```
{% endtab %}

{% tab title="Java" %}
The default synced Realm is provided via an automatic SyncConfiguration, which will use the current logged in user from `SyncUser.currentUser()` and the server URL used to authenticate.

For example, to log in, then asynchronously open the default synced Realm with the current user:

```java
SyncCredentials credentials = getCredentials();
String url = getUrl();
SyncUser.login(credentials, url, new SyncUser.Callback<SyncUser>() {
  @Override
  public void onSuccess(SyncUser user) {
    SyncConfiguration config = SyncConfiguration.automatic();
    Realm realm = Realm.getInstance(config);
    // Use Realm
  }

  @Override
  public void onError(ObjectServerError error) {
    // Handle error
  }
});
```

If you are working with multiple users, you can pass in the specific user as well:

```java
SyncUser user = getUser();
SyncConfiguration config = SyncConfiguration.automatic(user);
Realm realm = Realm.getInstance(config);
```
{% endtab %}

{% tab title="Javascript" %}
The default synced Realm is provided via the default configuration which will use the current logged in user from `Realm.Sync.User.current` and the server URL used to authenticate.

```javascript
Realm.Sync.User.login(server, username, password)
.then((user) => {
      let config = Realm.automaticSyncConfiguration();
      Realm.open(config).then((realm) => {
          // ...
      });
})
```

If you are working with multiple users, you can pass in the specific user as well:

```javascript
Realm.Sync.User.login(server, username, password)
.then((user) => {
      let config = Realm.automaticSyncConfiguration(user);
      Realm.open(config).then((realm) => {
          // ...
      });
})
```
{% endtab %}

{% tab title=".Net" %}
The default synced Realm is provided via an automatic `SyncConfiguration`, which will use the current logged in user from `User.Current` and the server URL used to authenticate.

For example, to log in, then asynchronously open the default synced Realm with the current user:

```csharp
var user = await User.LoginAsync(credentials, serverUrl);
RealmConfiguration.DefaultConfiguration = new SyncConfiguration();

var realm = await Realm.GetInstanceAsync();
```

If you are working with multiple users, you can pass in the specific user as well:

```csharp
var user = await User.LoginAsync(credentials, serverUrl);
RealmConfiguration.DefaultConfiguration = new SyncConfiguration(user);

var realm = await Realm.GetInstanceAsync();
```
{% endtab %}
{% endtabs %}

### Manually Configuring Synced Realms

If your application is using multiple Realms, you can manually configure a synced Realm by supplying:

1. An authenticated user
2. The Realm URL

{% hint style="info" %}
The Realm URL uses a specific scheme: `realm://` for non-secure connections and `realms://` for secure connections.

For more details, see the [Understanding Realm URLs and Paths](opening-a-synced-realm.md#understanding-realm-urls-and-paths) section below.
{% endhint %}

To obtain an authenticated user, you must login via any of the supported authentication providers. If you are unfamiliar, see the [Working With Users](user-authentication/) section.

{% tabs %}
{% tab title="Swift" %}
Realms on the Realm Object Server are using the same `Realm.Configuration` and factory methods that are used to create standalone Realms, but with the `syncConfiguration` property on their `Realm.Configuration` set to a `SyncConfiguration` value. Synchronized realms are located by [URLs](https://realm.io/docs/swift/latest/#server-url).

```swift
// Create the configuration
let syncServerURL = URL(string: "realm://localhost:9080/~/userRealm")!
let config = Realm.Configuration(syncConfiguration: SyncConfiguration(user: user, realmURL: syncServerURL))

// Open the remote Realm
let realm = try! Realm(configuration: config)
// Any changes made to this Realm will be synced across all devices!
```

The configuration values for a synced Realm cannot have an `inMemoryIdentifier` or `fileURL` configured. Setting either property will automatically nil out the `syncConfiguration` property \(and vice-versa\). The framework is responsible for managing how synchronized Realms are cached or stored on disk.
{% endtab %}

{% tab title="Objective-C" %}
Realms on the Realm Object Server are using the same `RLMRealmConfiguration` and factory methods that are used to create standalone Realms, but with the `syncConfiguration` property on their `RLMRealmConfiguration` set to a `RLMSyncConfiguration` value. Synchronized realms are located by [URLs](https://realm.io/docs/objc/latest/#server-url).

```objectivec
RLMSyncUser *user = [RLMSyncUser currentUser];

// Create the configuration
NSURL *syncServerURL = [NSURL URLWithString: @"realm://localhost:9080/~/userRealm"];
RLMRealmConfiguration *config = [RLMRealmConfiguration defaultConfiguration];
config.syncConfiguration = [[RLMSyncConfiguration alloc] initWithUser:user realmURL:syncServerURL];

// Open the remote Realm
RLMRealm *realm = [RLMRealm realmWithConfiguration:config error:nil];
// Any changes made to this Realm will be synced across all devices!
```

The configuration values for a synced Realm cannot have an `inMemoryIdentifier` or `fileURL` configured. Setting either property will automatically nil out the `syncConfiguration` property \(and vice-versa\). The framework is responsible for managing how synchronized Realms are cached or stored on disk.
{% endtab %}

{% tab title="Java" %}
Realms on the Realm Object Server are created using a subclass of the normal `RealmConfiguration` used to create standalone Realms. This class is named `SyncConfiguration` and uses the same builder pattern known from normal Realms. Specifically it requires an [User](user-authentication/) and an [URL](https://realm.io/docs/java/latest/#server-url).

```java
// Create the configuration
SyncUser user = SyncUser.currentUser();
String url = "realm://localhost:9080/~/userRealm";
SyncConfiguration config = new SyncConfiguration.Builder(user, url).build();

// Open the remote Realm
Realm realm = Realm.getInstance(config);
// Any changes made to this Realm will be synced across all devices!
```
{% endtab %}

{% tab title="Javascript" %}
You open a synchronized Realm the same say as you open any other Realm. The [configuration](https://realm.io/docs/javascript/latest/api/Realm.html#~Configuration) can be extended with a `sync` property if you need to configure the synchronization. The optional properties of `sync` include:

* `error` - a callback for error handling/reporting
* `validate_ssl` - indicating if SSL certificates must be validated
* `ssl_trust_certificate_path` - a path where to find trusted SSL certificates

The error handling is set up by registering a callback \(`error`\) as part of the configuration:

```javascript
const user = Realm.Sync.User.current;
const config = {
  sync: { user: user,
          url: "realm://localhost:9080/~/userRealm",
          error: err => console.log(err)
        },
  schema: // ...
};

var realm = new Realm(config);
```
{% endtab %}

{% tab title=".Net" %}
For standalone Realms, [RealmConfiguration](https://realm.io/docs/dotnet/latest/api/reference/Realms.RealmConfiguration.html) is used to configure the options for a Realm. Synchronized Realms on the other hand, are configured using an extended configuration class called [SyncConfiguration](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.SyncConfiguration.html).

The configuration ties together an authenticated user and a sync server URL. The sync server URL may contain the tilde character \(“~”\) which will be transparently expanded to represent the user’s unique identifier. This scheme easily allows you to write your app to cater to its individual users. The location on disk for shared Realms is managed by the framework, but can be overridden if desired.

```csharp
var user = User.Current;
var serverURL = new Uri("realm://my.realm-server.com:9080/~/default");
var configuration = new SyncConfiguration(user, serverURL);

var realm = Realm.GetInstance(configuration);
```
{% endtab %}
{% endtabs %}

### Partial Synchronization

If you intend to use partial synchronization, it is recommended you use the [Default Synced Realm](opening-a-synced-realm.md#the-default-synced-realm), which uses this by default. However, you are not limited to just this Realm. You can manually open additional Realms with partial synchronization by following the [manual process](opening-a-synced-realm.md#manually-configuring-synced-realms) described above, but adjusting another parameter in the sync configuration.

{% tabs %}
{% tab title="Swift" %}
```swift
const config = {
  sync: { user: user,
          url: realmUrl,
          partial: true,  // <-- this enables a partially synced Realm
        },
  // ...
};
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMRealmConfiguration *config = [RLMRealmConfiguration defaultConfiguration];
config.syncConfiguration = [[RLMSyncConfiguration alloc] initWithUser:user realmURL:realmURL];
config.syncConfiguration.isPartial = YES;
```
{% endtab %}

{% tab title="Java" %}
```java
SyncConfiguration config = new SyncConfiguration.Builder(getUser(), getUrl())
  .partialRealm()
  .build();
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
const config = {
  sync: { user: userA,
          url: realmUrl,
          partial: true,  // <-- this enables a partially synced Realm
        },
  // ...
};
```
{% endtab %}

{% tab title=".Net" %}
```csharp
var config = new SyncConfiguration(user, realmUrl)
{
    IsPartial = true // <-- this enables a partially synced Realm
};
```
{% endtab %}
{% endtabs %}

When you configure a Realm to use partial synchronization, it will initially have no data in it. For more information on how to sync data see the guide:

{% page-ref page="syncing-data.md" %}

## Synchronously Opening A Realm

To access a Realm immediately in your application, you can open a Realm "synchronously." This might be confusing since "synchronous" in this case means the API will return the Realm immediately. However, given that we are opening a _synchronized_ Realm, the first time you open the Realm there will always be no data in it. In the background, the Realm will establish a sync session and start downloading any existing data from the server. You can attach a progress listener to track the download activity or you can subscribe to notifications to get events when the data changes.

{% hint style="info" %}
This API is recommended when your want the user experience to not be blocked while the data is downloaded, such as displaying partial data to the user in the process.
{% endhint %}

{% tabs %}
{% tab title="Swift" %}
```swift
// Create the configuration
let syncServerURL = URL(string: "realm://localhost:9080/~/userRealm")!
let config = Realm.Configuration(syncConfiguration: SyncConfiguration(user: user, realmURL: syncServerURL))

// Open the remote Realm
let realm = try! Realm(configuration: config)
// Any changes made to this Realm will be synced across all devices!
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMSyncUser *user = [RLMSyncUser currentUser];

// Create the configuration
NSURL *syncServerURL = [NSURL URLWithString: @"realm://localhost:9080/~/userRealm"];
RLMRealmConfiguration *config = [RLMRealmConfiguration defaultConfiguration];
config.syncConfiguration = [[RLMSyncConfiguration alloc] initWithUser:user realmURL:syncServerURL];

// Open the remote Realm
RLMRealm *realm = [RLMRealm realmWithConfiguration:config error:nil];
// Any changes made to this Realm will be synced across all devices!
```
{% endtab %}

{% tab title="Java" %}
```java
// Create the configuration
SyncUser user = SyncUser.currentUser();
String url = "realm://localhost:9080/~/userRealm";
SyncConfiguration config = new SyncConfiguration.Builder(user, url).build();

// Open the remote Realm
Realm realm = Realm.getInstance(config);
// Any changes made to this Realm will be synced across all devices!
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
const user = Realm.Sync.User.current;
const config = {
  sync: { user: user,
          url: "realm://localhost:9080/~/userRealm",
          error: err => console.log(err)
        },
  schema: // ...
};

var realm = new Realm(config);
```
{% endtab %}

{% tab title=".Net" %}
```csharp
var user = User.Current;
var serverURL = new Uri("realm://my.realm-server.com:9080/~/default");
var configuration = new SyncConfiguration(user, serverURL);

var realm = Realm.GetInstance(configuration);
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
If a Realm has read-only [file-level permissions](access-control/path-level-permissions.md), then you _must_ asynchronously open the Realm as described in [Asynchronously Opening A Realm](opening-a-synced-realm.md#asynchronously-opening-a-realm). Opening a file-level read-only Realm without the asynchronous API will cause an error.

_This behavior does not apply to partially synced Realms that use the _[_finer-grainer access controls_](access-control/)_._
{% endhint %}

## Asynchronously Opening A Realm

In some cases, you might not want to open a Realm until it has all remote data available. For example, if you want to show the users a list of all available ZIP codes. Asynchronously opening a Realm uses an API that has a callback which will not return the Realm until it is fully downloaded.

{% tabs %}
{% tab title="Swift" %}
```swift
let config = Realm.Configuration(syncConfiguration: SyncConfiguration(user: user, realmURL: realmURL))
Realm.asyncOpen(configuration: config) { realm, error in
    if let realm = realm {
        // Realm successfully opened, with all remote data available
    } else if let error = error {
        // Handle error that occurred while opening or downloading the contents of the Realm
    }
}
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMRealmConfiguration *config = [RLMRealmConfiguration defaultConfiguration];
config.syncConfiguration = [[RLMSyncConfiguration alloc] initWithUser:user realmURL:realmURL];
[RLMRealm asyncOpenWithConfiguration:config
                       callbackQueue:dispatch_get_main_queue()
                            callback:^(RLMRealm *realm, NSError *error) {
    if (realm) {
        // Realm successfully opened, with all remote data available
    } else if (error) {
        // Handle error that occurred while opening or downloading the contents of the Realm
    }
}
```
{% endtab %}

{% tab title="Java" %}
```java
// Create the configuration specifying that the Realm cannot be opened
// the first time until server data has been downloaded. This only
// blocks it from being opened the first time. After that the Realm can
// be opened immediately.
SyncConfiguration config = new SyncConfiguration.Builder(user, url)
  .waitForInitialRemoteData();
  .build();

// Open the remote Realm
Realm realm = Realm.getInstanceAsync(config, new Realm.Callback() {
  @Override
  public void onSuccess(Realm realm) {
    // Realm is ready
  }

  @Override
  public void onError(Throwable exception) {
    // Handle error
  }
});
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
const user = Realm.Sync.User.current;
const config = {
  sync: { user: user,
          url: "realm://localhost:9080/~/userRealm",
          error: err => console.log(err)
        },
  schema: // ...
};

Realm.open(config)
  .then(realm => {
    // ...use the realm instance here
  })
  .catch(error => {
    // Handle the error here if something went wrong
  });
```
{% endtab %}

{% tab title=".Net" %}
```csharp
var serverURL = new Uri("realm://my.realm-server.com:9080/~/default");
var config = new SyncConfiguration(user, serverURL));
try
{
    var realm = await Realm.GetInstanceAsync(config);
    // Realm successfully opened, with all remote data available
}
catch (Exception ex)
{
    // Handle error that occurred while opening or downloading the contents of the Realm
}
```
{% endtab %}
{% endtabs %}

## Understanding Realm URLs and Paths

Synchronized Realms exists as resources tied to a specific URL path. If you are manually opening synchronized Realms and/or splitting your data up between multiple Realms then this is important to understand.

{% hint style="info" %}
If you are using the default synced Realm, you typically don't need to worry about this, as the Realm URL is automatically configured for you at the path: `/default`
{% endhint %}

### Realm Scheme

The URLs for synchronized Realms use a dedicated scheme. The underlying networking is not using standard HTTP, but instead Websockets. We chose not to use the Websockets scheme `ws`/`wss` because we utilize a custom protocol and wanted to emphasize the distinction.

For unsecured synchronized connections, the scheme is `realm://`.

For secured synchronized connections , the schema is `realms://`.

{% hint style="warning" %}
Take care not to mix up the `http`/`https` scheme used in the authentication URL when logging in a user.
{% endhint %}

### Path Restrictions

The Realm Object Server includes a path-level permission system. By default, it restricts the creation of Realms to scoped paths for the user. For example, if a user's ID is `12345` then this user can only create Realms at the path `/12345/myRealm`.

This restriction is not enforced for admin users. As a result, if you want to create a global Realm at the base path, `/globalRealm` the creation of the Realm must be done by an admin user.

You can learn more about the details of path-level permissions in its dedicated section, including how to adjust these permissions to share a Realm:

{% page-ref page="access-control/path-level-permissions.md" %}

### ~ Shorthand

When working with Realm paths you might quickly realize that it is time-consuming to keep track of the `userId` in a scoped-path. To simplify, Realm paths accept as shorthand the use of the tilde `~` character in exchange for the `userId`. For example, if you open a Realm with a `user` whose `userId` is `12345`, you can use either of these URL paths to open the same Realm:

```text
"realm://127.0.0.1:9080/~/myRealm"
```

```text
"realm://127.0.0.1:9080/12345/myRealm"
```

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3)

