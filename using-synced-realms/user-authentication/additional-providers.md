# Additional providers

## JWT

The JSON Web Token provider allows you to integrate with an existing authentication system. It requires that you modify the authentication server to expose a flow that produces signed JSON Web Tokens that your app then transmits to the Realm Object Server for verification.

### Configuration

{% tabs %}
{% tab title="Cloud" %}
{% hint style="warning" %}
JWT authentication is _disabled_ by default in Cloud as it requires configuration.
{% endhint %}

To enable or disable go to the Settings for your instance in the cloud portal:

![](../../.gitbook/assets/image%20%285%29.png)
{% endtab %}

{% tab title="Self-Hosted" %}
For more details on how to enable or disable this provider with Self-Hosted go to:

{% page-ref page="../../self-hosted/customize/authentication/jwt-custom-authentication.md" %}
{% endtab %}
{% endtabs %}

### Client API

{% tabs %}
{% tab title="Swift" %}
```swift
let jwtCredentials = SyncCredentials.jwt(token: "JWT token")
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMSyncCredentials *jwtCredentials = [RLMSyncCredentials credentialsWithJWT:@"JWT Token"];
```
{% endtab %}

{% tab title="Java" %}
```java
SyncCredentials credentials = SyncCredentials.jwt("my-jwt-token");
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
const jwtToken = 'acc3ssT0ken...';
Realm.Sync.User.registerWithProvider('http://my.realm-auth-server.com:9080', 'jwt', jwtToken, (error, user) => { /* ... */ });
```
{% endtab %}

{% tab title=".Net" %}
```csharp
var jwtToken = "...";
var credentials = Credentials.Custom("jwt", jwtToken, null);
```
{% endtab %}
{% endtabs %}

## Anonymous

Sometimes it's useful to create a synchronized Realm for a user without prompting them for credentials - e.g. in an e-commerce application where any disruption leads to user churn. For such cases, the Realm Object Server exposes an anonymous provider that requires no payload and creates a new user for every request \(which means that you must use the built-in client-side user caching to avoid resetting the app state on every launch\).

### Configuration

{% tabs %}
{% tab title="Cloud" %}
{% hint style="success" %}
By default anonymous authentication is enabled with Cloud
{% endhint %}

To enable or disable go to the Settings for your instance in the cloud portal:

![](../../.gitbook/assets/image%20%281%29.png)
{% endtab %}

{% tab title="Self-Hosted" %}
{% hint style="success" %}
By default the `AnonymousAuthProvider` is **enabled** when creating a project via `ros init`.
{% endhint %}

For more details on how to enable or disable this provider with Self-Hosted go to:

{% page-ref page="../../self-hosted/customize/authentication/anonymous-authentication.md" %}
{% endtab %}
{% endtabs %}

### Client API

{% tabs %}
{% tab title="Swift" %}
```swift
let anonymousCredentials = SyncCredentials.anonymous()
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMSyncCredentials *anonymousCredentials = [RLMSyncCredentials anonymousCredentials];
```
{% endtab %}

{% tab title="Java" %}
```
SyncCredentials credentials = SyncCredentials.anonymous();
```
{% endtab %}

{% tab title="Javascript" %}
{% hint style="warning" %}
_API Coming Soon!_
{% endhint %}
{% endtab %}

{% tab title=".Net" %}
{% hint style="warning" %}
_API Coming Soon!_
{% endhint %}
{% endtab %}
{% endtabs %}



## Nickname

The Nickname provider allows you to authenticate users just by their nickname without requiring passwords or other identifying information. This is useful when prototyping collaborative features for your app without worrying about the actual login flow until later in the development lifecycle.

{% hint style="danger" %}
The Nickname provider is not secure and should never be enabled in production deployments. It's meant to only be used during development.
{% endhint %}

### Configuration

{% tabs %}
{% tab title="Cloud" %}
{% hint style="warning" %}
By default Nickname authentication is enabled with Cloud
{% endhint %}

To enable or disable go to the Settings for your instance in the cloud portal:

![](../../.gitbook/assets/image%20%2819%29.png)
{% endtab %}

{% tab title="Self-Hosted" %}
{% hint style="warning" %}
By default the Nickname provider is **enabled** when creating a project via `ros init`.
{% endhint %}

For more details on how to enable or disable this provider with Self-Hosted go to:

{% page-ref page="../../self-hosted/customize/authentication/nickname-authentication.md" %}
{% endtab %}
{% endtabs %}

### Client API

{% tabs %}
{% tab title="Swift" %}
```swift
let nicknameCredentials = SyncCredentials.nickname(nickname: "Adam")
```

To create an admin user:

```swift
let nicknameCredentials = SyncCredentials.nickname(nickname: "Adam" isAdmin: true)
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMSyncCredentials *nicknameCredentials = [RLMSyncCredentials credentialsWithNickname:@"Adam"
                                                                              isAdmin:NO];
```

To create an admin user:

```objectivec
RLMSyncCredentials *nicknameCredentials = [RLMSyncCredentials credentialsWithNickname:@"Adam"
                                                                              isAdmin:YES];
```
{% endtab %}

{% tab title="Java" %}
```java
SyncCredentials credentials = SyncCredentials.nickname("my-nickname", false);
```

The last `boolean` indicates if the user created should be an admin user or not.
{% endtab %}

{% tab title="Javascript" %}
{% hint style="warning" %}
_API Coming Soon!_
{% endhint %}
{% endtab %}

{% tab title=".Net" %}
{% hint style="warning" %}
_API Coming Soon!_
{% endhint %}
{% endtab %}
{% endtabs %}



## Custom Authentication

{% hint style="warning" %}
This provider is not available in Realm Platform - Cloud
{% endhint %}

With Realm Platform - Self-Hosted you have the ability to directly customize the authentication code run by the server. This allows for any form of custom authentication and unlike JWT, it does not require any changes to an existing authentication system.

### Configuration

For more details on how to use the custom authentication, see the Self-Hosted docs:

{% page-ref page="../../self-hosted/customize/authentication/custom-authentication.md" %}

### Client API

{% tabs %}
{% tab title="Swift" %}
```swift
let customCredentials = SyncCredentials(customToken: "token", provider: Provider("myauth"))
```

Additional information can be passed to the custom credentials initializer as a third parameter. Please see the [API reference](https://realm.io/docs/swift/latest/api/Structs/SyncCredentials.html#/s:ZFV10RealmSwift15SyncCredentials3jwtFSSS0_) for more information.
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMSyncCredentials *credentials = [[RLMSyncCredentials alloc] initWithCustomToken:@"custom token" provider:@"myauth" userInfo:nil];
```
{% endtab %}

{% tab title="Java" %}
```java
String token = "..."; // a string representation of a token obtained from your authentication server
Map<String, Object> customData = new HashMap<>();
SyncCredentials myCredentials = SyncCredentials.custom(
  token,
  'myauth',
  customData,
);
```

_Note:_ it is possible to pass additional login information to this constructor as a third parameter. Please see the [SyncCredentials API](https://realm.io/docs/java/latest/api/io/realm/SyncCredentials.html#custom-java.lang.String-java.lang.String-java.util.Map-) for more information.
{% endtab %}

{% tab title="Javascript" %}
```javascript
// The user token provided by your authentication server
const accessToken = 'acc3ssT0ken...';

const user = Realm.Sync.User.registerWithProvider(
  'http://my.realm-auth-server.com:9080',
  'my-custom-provider',
  accessToken,
  (error, user) => { /* ... */ }
);
```

_Note:_ the JavaScript SDK does not currently allow you to send additional data. If you need to send more than a single token, please encode the additional data as JSON and pass it through the accessToken parameter, and decode this string on the server side.
{% endtab %}

{% tab title=".Net" %}
```aspnet
let customCredentials = SyncCredentials(customToken: "token", provider: Provider("myauth"))
```

Additional information can be passed to a custom auth provider in the form of a dictionary provided as the third parameter; see the [API Reference](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.Credentials.html) for details.
{% endtab %}
{% endtabs %}

## Google

{% hint style="warning" %}
This provider is not available in Realm Platform - Cloud
{% endhint %}

The Realm Object Server package includes a pre-built provider for Google. Clients authenticate through Google SDK/API and sends the access token to ROS. This provider handles validating the access token with Google’s API, and then authenticating in Realm.

### Configuration

For more details on how to use the Google provider, see the Self-Hosted docs:

{% page-ref page="../../self-hosted/customize/authentication/included-third-party-auth-providers/google-authentication.md" %}

### Client API

{% tabs %}
{% tab title="Swift" %}
```swift
let googleCredentials = SyncCredentials.google(token: "Google token")
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMSyncCredentials *googleCredentials = [RLMSyncCredentials credentialsWithGoogleToken:@"Google token"];
```
{% endtab %}

{% tab title="Java" %}
```java
SyncCredentials credentials = SyncCredentials.google("my-google-token");
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
const googleAccessToken = 'acc3ssT0ken...';
Realm.Sync.User.registerWithProvider('http://my.realm-auth-server.com:9080', 'google', googleAccessToken, (error, user) => { /* ... */ });
```
{% endtab %}

{% tab title=".Net" %}
```csharp
var token = "..."; // a string representation of a token obtained by Google Login API
var credentials = Credentials.Google(token);
```
{% endtab %}
{% endtabs %}

## Facebook

{% hint style="warning" %}
This provider is not available in Realm Platform - Cloud
{% endhint %}

The Realm Object Server package includes a pre-built provider for Facebook. Clients authenticate through Facebook SDK/API and sends the access token to ROS. This provider handles validating the access token with Facebook’s API, and then authenticating in Realm.

### Configuration

For more details on how to use the Google provider, see the Self-Hosted docs:

{% page-ref page="../../self-hosted/customize/authentication/included-third-party-auth-providers/facebook-authentication.md" %}

### Client API

{% tabs %}
{% tab title="Swift" %}
```swift
let facebookCredentials = SyncCredentials.facebook(token: "Facebook token")
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMSyncCredentials *facebookCredentials = [RLMSyncCredentials credentialsWithFacebookToken:@"Facebook token"];
```
{% endtab %}

{% tab title="Java" %}
```java
SyncCredentials credentials = SyncCredentials.facebook("my-facebook-token");
```
{% endtab %}

{% tab title="Javascript" %}
```javascript
const fbAccessToken = 'acc3ssT0ken...';
Realm.Sync.User.registerWithProvider('http://my.realm-auth-server.com:9080', 'facebook', fbAccessToken, (error, user) => { /* ... */ });
```
{% endtab %}

{% tab title=".Net" %}
```csharp
var token = "..."; // a string representation of a token obtained by Facebook Login API
var credentials = Credentials.Facebook(token);
```
{% endtab %}
{% endtabs %}

## Azure Active Directory

{% hint style="warning" %}
This provider is not available in Realm Platform - Cloud
{% endhint %}

The Realm Object Server package includes a pre-built provider for Azure Active Directory. Clients authenticate through Azure SDK/API and sends the access token to ROS. This provider handles validating the JWT access token and then authenticating in Realm.

### Configuration

For more details on how to use the Azure Active Directory provider, see the Self-Hosted docs:

{% page-ref page="../../self-hosted/customize/authentication/included-third-party-auth-providers/azure-authentication.md" %}

### Client API

{% tabs %}
{% tab title=".Net" %}
```aspnet
var token = "..."; // a string representation of a token obtained by logging in with Azure Active Directory
var credentials = Credentials.AzureAD(token);
```
{% endtab %}
{% endtabs %}

## CloudKit

{% hint style="warning" %}
This provider is not available in Realm Platform - Cloud
{% endhint %}

The Realm Object Server package includes a pre-built provider for Apple's CloudKit. Clients authenticate through CloudKit API and retrieve a token which is passed to ROS to identify and authenticate the user.

### Configuration

For more details on how to use the Apple CloudKit provider, see the Self-Hosted docs:

{% page-ref page="../../self-hosted/customize/authentication/included-third-party-auth-providers/cloudkit-authentication.md" %}

### Client API

{% tabs %}
{% tab title="Swift" %}
```swift
let cloudKitCredentials = SyncCredentials.cloudKit(token: "CloudKit token")
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMSyncCredentials *cloudKitCredentials = [RLMSyncCredentials credentialsWithCloudKitToken:@"CloudKit token"];
```
{% endtab %}
{% endtabs %}







