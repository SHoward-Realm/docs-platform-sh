# User Authentication

The central object in the Realm Object Server is the Realm User \(`SyncUser`\). A Realm User is used to identify the end-user of the application and is used with the server's [access control](../access-control/) functionality to securely control what data is synchronized.  A `SyncUser` [authenticates](./#login), or performs a [login](./#login), via a username/password scheme, or through a number of third-party authentication methods.

Creating and logging in a user requires two things:

* A URL of a Realm Object Server to connect to.
* Credentials for an authentication mechanism that describes the user as appropriate for that mechanism \(i.e., username/password, access key, etc\).

## Server URL

To authenticate, you must supply a server URL. This is the base URL for your server, such as `https://myinstance.cloud.realm.io` or `http://127.0.0.1:9080`.

{% hint style="warning" %}
Note this URL uses the `http`/`https` scheme as user authentication is handled via standard HTTP. Be careful not to confuse this with the Realm URL and its `realm`/`realms` scheme.
{% endhint %}

{% tabs %}
{% tab title="Swift" %}
The _authentication server URL_ is simply a `URL` representing the location of the Realm Object Server.

```swift
let authURL = URL(string: "https://myinstance.cloud.realm.io")!
```
{% endtab %}

{% tab title="Objective-C" %}
The _authentication server URL_ is simply a `NSURL` representing the location of the Realm Object Server.

```objectivec
NSURL *authURL = [NSURL URLWithString:@"https://myinstance.cloud.realm.io"];
```
{% endtab %}

{% tab title="Java" %}
The _authentication server URL_ is simply a `URL` representing the location of the Realm Object Server.

```java
String authURL = "https://myinstance.cloud.realm.io"
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
const authURL = "https://myinstance.cloud.realm.io";
```
{% endtab %}

{% tab title=".Net" %}
```csharp
var authURL = new Uri("https://myinstance.cloud.realm.io");
```
{% endtab %}
{% endtabs %}

## Credentials

The _credential information_ for a given user is represented by a `SyncCredentials` value, which can be created in one of several ways:

* Providing a valid username/password combination
* Providing a token obtained from a supported third-party authentication service
* Providing a token and a custom authentication provider \(see our documentation on [custom authentication](https://realm.io/docs/realm-object-server/#custom-authentication)\)

The username and password authentication is entirely managed by the Realm Object Server, giving you full control over your application’s user management. For other authentication methods, your application is responsible for logging into the external service and obtaining the authentication token.

Here are some examples of setting credentials with various providers.

### Username/password

{% tabs %}
{% tab title="Swift" %}
```swift
let usernameCredentials = SyncCredentials.usernamePassword(username: "username", password: "password")
```

Note that the factory method takes a `register` boolean argument that indicates whether a new user should be registered or an existing user should be logged in. An error will be thrown if your application tries to register a new user with the username of an existing user, or tries to log in a user that does not exist.
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMSyncCredentials *usernameCredentials = [RLMSyncCredentials credentialsWithUsername:@"username"
                                                                             password:@"password"
                                                                             register:NO];
```

Note that the factory method takes a `register` boolean argument that indicates whether a new user should be registered or an existing user should be logged in. An error will be thrown if your application tries to register a new user with the username of an existing user, or tries to log in a user that does not exist.
{% endtab %}

{% tab title="Java" %}
```java
String username = "jane"
String password = "doe"
boolean createUser = true;
SyncCredentials credentials = SyncCredentials.usernamePassword(username, password, createUser);
```

Note that the factory method takes a `createUser` boolean argument that indicates whether a new user should be registered and created or an existing user should be logged in. An error will be thrown if your application tries to register a new user with the username of an existing user, or tries to log in a user that does not exist.
{% endtab %}

{% tab title="Javascript" %}
```javascript
Realm.Sync.User.login('http://my.realm-auth-server.com:9080', 'username', 'p@s$w0rd').then(user => {
  // user is logged in
  // do stuff ...
}).catch(error => {
  // an auth error has occurred
});
```

Before a user can log in, the account must be created. You can either do that in advance on the server using the admin dashboard, or by calling `register`:

```javascript
Realm.Sync.User.register('http://my.realm-auth-server.com:9080', 'username', 'p@s$w0rd', (error, user) => { /* ... */ });
```
{% endtab %}

{% tab title=".Net" %}
```csharp
var credentials = Credentials.UsernamePassword(username, password, createUser: false);
```

The third parameter of [UsernamePassword\(\)](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.Credentials.html#Realms_Sync_Credentials_UsernamePassword_System_String_System_String_System_Boolean_) is the `createUser` flag which should only be `true` the first time, as it indicates that the user should be created. Once the user is created, the parameter must be set to `false`.

Alternatively, you can require that all users are created in advance on the server using an admin Dashboard, and always pass `false` for the `createUser` parameter.
{% endtab %}
{% endtabs %}

### Custom Authentication

Realm Platform supports the ability to use external authentication providers. This allow users to authenticate users against legacy databases or APIs, or integrate with providers that are not supported out-of-the-box.

For example, Realm Platform includes a JWT authentication provider. It requires that you modify your own authentication server to expose a flow that produces signed JSON Web Tokens that your app then transmits to the Realm Object Server for verification. The JSON Web Token would be sent from the client application via a custom authentication credential as shown below:

{% tabs %}
{% tab title="Swift" %}
```swift
let customCredentials = SyncCredentials(customToken: "JSON Web Token", provider: Provider("jwt"))
```

Additional information can be passed to the custom credentials initializer as a third parameter. Please see the [API reference](https://realm.io/docs/swift/2.4.3/api/Structs/SyncCredentials.html#/s:FV10RealmSwift15SyncCredentialscFT11customTokenSS8providerVSC19RLMIdentityProvider8userInfoGVs10DictionarySSP___S0_) for more information.
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMSyncCredentials *credentials = [[RLMSyncCredentials alloc] initWithCustomToken:@"JSON Web Token" provider:@"jwt" userInfo:nil];
```

Additional information may be passed to the custom credentials constructor as a third parameter. Please see the [API reference](https://realm.io/docs/objc/2.4.3/api/Classes/RLMSyncCredentials.html#/c:objc%28cs%29RLMSyncCredentials%28im%29initWithCustomToken:provider:userInfo:) for more information.
{% endtab %}

{% tab title="Java" %}
```java
String token = "..."; // Custom token
String authProvider = "custom-provider"; // Name of custom provider
Map<String, Object> userInfo = null; // Custom data or null
SyncCredentials credentials = SyncCredentials.custom(token, authProvider, userInfo);
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
// The user token provided by your authentication server
const accessToken = 'acc3ssT0ken...';

const user = Realm.Sync.User.registerWithProvider(
  'http://my.realm-auth-server.com:9080',
  'custom/fooauth',
  accessToken,
  (error, user) => { /* ... */ }
);
```

_Note:_ the JavaScript SDK does not currently allow you to send additional data. If you need to send more than a single token, please encode the additional data as JSON and pass it through the accessToken parameter, and decode this string on the server side.
{% endtab %}

{% tab title=".Net" %}
```csharp
var token = "..."; // a string representation of a token obtained from your custom provider
var credentials = Credentials.Custom("myAuth", token, null);
```

Additional information can be passed to a custom auth provider in the form of a dictionary provided as the third parameter; see the [API Reference](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.Credentials.html) for details.
{% endtab %}
{% endtabs %}

### Other

Additional authentication providers are available with the self-hosted version. For more details see:

{% page-ref page="additional-providers.md" %}

## Login

{% tabs %}
{% tab title="Swift" %}
To create a user, call `SyncUser.logIn()`. This factory method will initialize the user, asynchronously log them in to the Realm Object Server, and then provide the user object for further use in a callback block if the login process is successful. If the login process fails for any reason, an error object will be passed into the callback instead.

```swift
SyncUser.logIn(with: credentials,
               server: serverURL) { user, error in
    if let user = user {
        // can now open a synchronized Realm with this user
    } else if let error = error {
        // handle error
    }
}
```
{% endtab %}

{% tab title="Objective-C" %}
To create a user, call `+[RLMSyncUser logIn]`. This factory method will initialize the user, asynchronously log them in to the Realm Object Server, and then provide the user object for further use in a callback block if the login process is successful. If the login process fails for any reason, an error object will be passed into the callback instead.

```objectivec
[RLMSyncUser logInWithCredentials:usernameCredentials
                    authServerURL:serverURL
                     onCompletion:^(RLMSyncUser *user, NSError *error) {
    if (user) {
        // can now open a synchronized RLMRealm with this user
    } else if (error) {
        // handle error
    }
}];
```
{% endtab %}

{% tab title="Java" %}
To create a user, call `SyncUser.login()` or `SyncUser.loginAsync().`Once the login request succeed, the user has logged into the Realm Object Server and is ready to be used to open Realms. Since this involves a network request, the synchronous method should only be used on background threads.

```java
SyncCredentials credentials = getCredentials();
String url = "https://my.realm.instance"
SyncUser.loginAsync(credentials, url, new SyncUser.Callback<SyncUser>() {
  @Override
  public void onSuccess(SyncUser user) {
    // User is ready
    // Can also be accessed using `SyncUser.currentUser()` if only one 
    // user is logged in.
  }
  
  @Override
  public void onError(ObjectServerError error) {
    // Something went wrong
  }
});
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
Realm.Sync.User.login('http://my.realm-auth-server.com:9080', 'username', 'p@s$w0rd').then(user => {
  // user is logged in
  // do stuff ...
}).catch(error => {
  // an auth error has occurred
});
```

Before a user can log in, the account must be created. You can either do that in advance on the server using the admin dashboard, or by calling `register`:

```text
Realm.Sync.User.register('http://my.realm-auth-server.com:9080', 'username', 'p@s$w0rd', (error, user) => { /* ... */ });
```
{% endtab %}

{% tab title=".Net" %}
```csharp
var authURL = new Uri("http://my.realm-auth-server.com:9080");
var user = await User.LoginAsync(credentials, authURL);
```
{% endtab %}
{% endtabs %}

## Logout

{% tabs %}
{% tab title="Swift" %}
To log a user out of their account, call `SyncUser.logOut()`. Any pending local changes will continue to be uploaded until the Realm Object Server has been fully synchronized. Then, all of their local synced Realms will be deleted from their device on next app launch.
{% endtab %}

{% tab title="Objective-C" %}
To log a user out of their account, call `[RLMSyncUser logOut]`. Any pending local changes will continue to be uploaded until the Realm Object Server has been fully synchronized. Then, all of their local synced Realms will be deleted from their device on next app launch.
{% endtab %}

{% tab title="Java" %}
To log a user out of their account, call `SyncUser.logout()`. Any pending local changes will continue to be uploaded until the Realm Object Server has been fully synchronized. Then, all of their local synced Realms will be deleted from their device on next app launch.
{% endtab %}

{% tab title="Javascript" %}
```javascript
user.logout();
```

When a user is logged out, the synchronization will stop. A logged out user can no longer open a synced Realm.
{% endtab %}

{% tab title=".Net" %}
```csharp
user.LogOut();
```

When a user is logged out, the synchronization will stop. A logged out user can no longer open a Realm using a [SyncConfiguration](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.SyncConfiguration.html).
{% endtab %}
{% endtabs %}

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

