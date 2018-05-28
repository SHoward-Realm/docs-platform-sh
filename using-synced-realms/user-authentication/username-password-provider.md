# Username/password provider

## Overview

Realm Platform provides a built-in username and password authentication provider. This allows client applications to register users with unique usernames and offers customizable password handling.

### Configuration

{% tabs %}
{% tab title="Cloud" %}
Realm Cloud is configured to send emails in order to allow users to complete a password reset or email confirmation flow. This is enabled by default, however, if you want to override this configuration \(example for strong branding\) you can specify a custom SMTP and email sender. 

* Navigate to your instance settings

![](../../.gitbook/assets/screen-shot-2018-04-25-at-12.38.25.png)

* Select the Username / Password provider, then check "send emails" to activate the option.

![](../../.gitbook/assets/screen-shot-2018-04-25-at-15.31.17.png)

You can now keep using the default configuration to send emails, or continue for further customization.  

* Uncheck the default configuration, to override it 

![](../../.gitbook/assets/screen-shot-2018-04-25-at-13.32.12.png)
{% endtab %}

{% tab title="Self-Hosted" %}
{% page-ref page="../../self-hosted/customize/authentication/username-password/password-reset-and-email-confirmation.md" %}
{% endtab %}
{% endtabs %}

## Register Users

{% tabs %}
{% tab title="Swift" %}
```swift
let usernameCredentials = SyncCredentials.usernamePassword(username: "username", password: "password", register: true)
```

Note that the factory method takes a `register` boolean argument that indicates whether a new user should be registered or an existing user should be logged in. An error will be thrown if your application tries to register a new user with the username of an existing user, or tries to log in a user that does not exist.
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMSyncCredentials *usernameCredentials = [RLMSyncCredentials credentialsWithUsername:@"username"
                                                                             password:@"password"
                                                                             register:YES];
```

Note that the factory method takes a `register` boolean argument that indicates whether a new user should be registered or an existing user should be logged in. An error will be thrown if your application tries to register a new user with the username of an existing user, or tries to log in a user that does not exist.
{% endtab %}

{% tab title="Java" %}
```java
SyncCredentials myCredentials = SyncCredentials.usernamePassword(username, password, true);
```

The third parameter of `usernamePassword` is a boolean indicating whether the user should be created. Set it to `true` when registering new users; set it to `false` when logging in users.  
{% endtab %}

{% tab title="Javascript" %}
```javascript
Realm.Sync.User.register('http://my.realm-auth-server.com:9080', 'username', 'p@s$w0rd', (error, user) => { /* ... */ });
```
{% endtab %}

{% tab title=".Net" %}
```csharp
var credentials = Credentials.UsernamePassword(username, password, createUser: true);
```

The third parameter of [UsernamePassword\(\)](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.Credentials.html#Realms_Sync_Credentials_UsernamePassword_System_String_System_String_System_Boolean_) is the `createUser` flag which should only be `true` the first time, as it indicates that the user should be created. Once the user is created, the parameter must be set to `false`.
{% endtab %}
{% endtabs %}

## Login Users

{% tabs %}
{% tab title="Swift" %}
```swift
let auth_url = URL(string: "https://myinstance.realm.io")!
let creds    = SyncCredentials.usernamePassword(username: "username", password: "password", register: true)

SyncUser.logIn(with: creds, server: auth_url, onCompletion: { [weak self](user, err) in
    if let _ = user {
        // User is logged in
    } else if let error = err {
        fatalError(error.localizedDescription)
    }
})
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
RLMSyncCredentials *usernameCredentials = [RLMSyncCredentials credentialsWithUsername:@"username"
                                                                             password:@"password"
                                                                             register:NO];
```
{% endtab %}

{% tab title="Java" %}
```java
SyncCredentials myCredentials = SyncCredentials.usernamePassword(username, password, false);
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
{% endtab %}

{% tab title=".Net" %}
```csharp
var credentials = Credentials.UsernamePassword(username, password, createUser: false);
```
{% endtab %}
{% endtabs %}

## Change Password

{% tabs %}
{% tab title="Swift" %}
### Change Current User's Password {#change-current-user's-password}

Users who authenticate using the built-in Realm Object Server username/password credential type may change their own passwords by calling the `SyncUser.changePassword(_:, completion:)` API.

This API makes an asynchronous call to the server. The completion block will be called once a response is received, and will be passed in an error if the operation failed, or nil if the operation succeeded.

```swift
let newPassword = "swordfish"
user.changePassword(newPassword) { (error) in
    if let error = error {
        // Something went wrong
    }
    // Otherwise, the password was successfully changed.
}
```

### Change Another User's Password {#change-another-user's-password}

Administrators may change the password of any user by calling the `SyncUser.changePassword(_:, forUserID:, completion:)` API. Pass in the user identity of the user whose password should be changed.

```swift
let newPassword = "swordfish"
adminUser.changePassword(newPassword, forUserID: "12345...") { (error) in
    if let error = error {
        // Something went wrong
    }
    // Otherwise, the password was successfully changed.
}
```
{% endtab %}

{% tab title="Objective-C" %}
### Change Current User's Password

Users who authenticate using the built-in Realm Object Server username/password credential type may change their own passwords by calling the `-[RLMSyncUser changePassword:completion:]` API.

This API makes an asynchronous call to the server. The completion block will be called once a response is received, and will be passed in an error if the operation failed, or nil if the operation succeeded.

```objectivec
NSString *newPassword = @"swordfish";
[user changePassword:newPassword
          completion:^(NSError *error) {
    if (error) {
        // Something went wrong
    }
    // Otherwise, the password was successfully changed.
}];
```

### Change Another User's Password {#change-another-user's-password}

Administrators may change the password of any user by calling the `-[RLMSyncUser changePassword:forUserID:completion:]` API. Pass in the user identity of the user whose password should be changed.

```objectivec
NSString *newPassword = @"swordfish";
[adminUser changePassword:newPassword
                forUserId:@"12345..."
               completion:^(NSError *error) {
    if (error) {
        // Something went wrong
    }
    // Otherwise, the password was successfully changed.
}];
```
{% endtab %}

{% tab title="Java" %}
Users who authenticate using the built-in Realm Object Server username/password credential type may change their own passwords by calling the either the `SyncUser.changePassword()` or `SyncUser.changePasswordAsync()`API.

This API makes a call to the server. Once the method returns, the user will have updated their password. If any error happens, either an exception is thrown or it is delivered in the callback.

```java
SyncUser user = SyncUser.currentUser();

// Change password on a background thread
try {
  user.changePassword("new-password);
  // Password was successfully changed
} catch (ObjectServerError error) {
  // Something went wrong
}

// Change password from the UI thread
user.changePasswordAsync("new-password", new SyncUser.Callback<SyncUser>() {
  @Override
  public void onSuccess(SyncUser user) {
    // Password for 'user' was succesfully changed.
  }
  
  @Override
  public void onError(ObjectServerError error) {
    // Something went wrong
  }
});
```
{% endtab %}

{% tab title="Javascript" %}
{% hint style="info" %}
_API Coming Soon!_
{% endhint %}
{% endtab %}

{% tab title=".Net" %}
To change a user’s password, you can use [User.ChangePasswordAsync](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.User.html#Realms_Sync_User_ChangePasswordAsync_System_String_):

```csharp
var currentUser = User.Current;
await currentUser.ChangePasswordAsync("new-secure-password");

// The user's password has been changed, they'll no longer be able to use their old password.
```

Admin users may change other users’ passwords \(e.g. as part of a server-side app that automates password retrieval\) by passing in a user’s Id:

```csharp
var adminUser = await User.LoginAsync(...);
await adminUser.ChangePasswordAsync("some-user-id", "new-secure-password");

// The password of the user with Id some-user-id has been changed to new-secure-password.
```
{% endtab %}
{% endtabs %}

## Reset Password

Users who authenticate using the built-in Realm Object Server username/password credential type may initiate a password reset if they used a valid email as their username, this is done in two steps:

1. Request a password reset: 

{% tabs %}
{% tab title="Swift" %}
```swift
let serverURL = URL(string: "https://foo.us1.realm.cloud.io")
let email = "user@email.com"
SyncUser.requestPasswordReset(forAuthServer: serverURL, userEmail: email) { error in
    if error {
        // something went wrong
    }
}
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
NSURL *serverURL = [NSURL URLWithString:@"https://foo.us1.realm.cloud.io"];
NSString *email = @"user@email.com";
[RLMSyncUser requestPasswordResetForAuthServer:serverURL userEmail:email completion:^(NSError *error) {
    if (error) {
        // Something went wrong
    }
}];
```
{% endtab %}

{% tab title="Java" %}
```java
SyncUser.requestPasswordResetAsync("user@email.com", "https://foo.us1.realm.cloud.io", new SyncUser.Callback<Void>() {
    @Override
    public void onSuccess(Void result) {        
    }

    @Override
    public void onError(ObjectServerError error) {
    }
});
```
{% endtab %}

{% tab title="Javascript" %}
```typescript
Realm.Sync.User.requestPasswordReset('https://foo.us1.realm.cloud.io', 'user@email.com').then(() => {
    // query sent successfully. 
}).catch((error) => {
    // an error has occurred
});
```
{% endtab %}

{% tab title=".NET" %}
```csharp
var serverUri = new Uri("https://foo.us1.realm.cloud.io");
await User.RequestPasswordResetAsync(serverUri, "user@email.com");
```
{% endtab %}
{% endtabs %}

   2. Complete the password reset using the token sent by email, this can be completed by clicking on the link inside the email or via the API if we manage to extract the token from the email \(using [deep linking](https://en.wikipedia.org/wiki/Mobile_deep_linking) for instance\):

{% tabs %}
{% tab title="Swift" %}
```swift
let serverURL = URL(string: "https://foo.us1.realm.cloud.io")
let token = "resetToken"
let newPassword = "newPassword"
SyncUser.completePasswordReset(forAuthServer: serverURL, token: token, password: newPassword) { error in
    if error {
        // something went wrong
    }
}
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
NSURL *serverURL = [NSURL URLWithString:@"https://foo.us1.realm.cloud.io"];
NSString *token = @"reset_token";
NSString *newPassword = @"newPassword";
[RLMSyncUser completePasswordResetForAuthServer:serverURL token:token password:newPassword completion:^(NSError *error) {
    if (error) {
        // Something went wrong
    }
}];
```
{% endtab %}

{% tab title="Java" %}
```java
SyncUser.completePasswordResetAsync("reset_token", "newPassword", "https://foo.us1.realm.cloud.io", new SyncUser.Callback<Void>() {
    @Override
    public void onSuccess(Void result) {
    }

    @Override
    public void onError(ObjectServerError error) {
    }
});
```
{% endtab %}

{% tab title="Javascript" %}
```typescript
Realm.Sync.User.completePasswordReset('https://foo.us1.realm.cloud.io', 'reset_token', 'new_password').then(() => {
    // query sent successfully. 
}).catch((error) => {
    // an error has occurred
});
```
{% endtab %}

{% tab title=".NET" %}
```csharp
var serverUri = new Uri("https://foo.us1.realm.cloud.io");
await User.CompletePasswordResetAsync(serverUri, "reset_token", "newPassword");
```
{% endtab %}
{% endtabs %}

## Confirming email 

After registering a new user, we can perform email confirmation if the username used is a valid email, this is done in two steps:

1. Request email confirmation

{% tabs %}
{% tab title="Swift" %}
```swift
let serverURL = URL(string: "https://foo.us1.realm.cloud.io")
let email = "user@email.com"
SyncUser.requestEmailConfirmation(forAuthServer: serverURL, userEmail: email) { error in
    if error {
        // something went wrong
    }
}
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
NSURL *serverURL = [NSURL URLWithString:@"https://foo.us1.realm.cloud.io"];
NSString *email = @"user@email.com";
[RLMSyncUser requestEmailConfirmationForAuthServer:serverURL userEmail:email completion:^(NSError *error) {
    if (error) {
        // Something went wrong
    }
}];
```
{% endtab %}

{% tab title="Java" %}
```java
SyncUser.requestEmailConfirmationAsync("user@email.com", "https://foo.us1.realm.cloud.io", new SyncUser.Callback<Void>() {
    @Override
    public void onSuccess(Void result) {
    }

    @Override
    public void onError(ObjectServerError error) {
    }
});
```
{% endtab %}

{% tab title="Javascript" %}
```typescript
Realm.Sync.User.requestEmailConfirmation('https://foo.us1.realm.cloud.io', 'user@email.com').then(() => {
    // query sent successfully. 
}).catch((error) => {
    // an error has occurred
});
```
{% endtab %}

{% tab title=".NET" %}
```csharp
var serverUri = new Uri("https://foo.us1.realm.cloud.io");
await User.RequestEmailConfirmationAsync(serverUri, "user@email.com");
```
{% endtab %}
{% endtabs %}

   2. Complete the email confirmation using the token sent by email,  this can be completed by clicking on the link inside the email or via the API if we manage to extract the token from the email \(using [deep linking](https://en.wikipedia.org/wiki/Mobile_deep_linking) for instance\):

{% tabs %}
{% tab title="Swift" %}
```swift
let serverURL = URL(string: "https://foo.us1.realm.cloud.io")
let token = "resetToken"
SyncUser.confirmEmail(forAuthServer: serverURL, token: token) { error in
    if error {
        // something went wrong
    }
}
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
NSURL *serverURL = [NSURL URLWithString:@"https://foo.us1.realm.cloud.io"];
NSString *token = @"confirmation_token";
[RLMSyncUser confirmEmailForAuthServer:serverURL token:token completion:^(NSError *error) {
    if (error) {
        // Something went wrong
    }
}];
```
{% endtab %}

{% tab title="Java" %}
```java
SyncUser.confirmEmailAsync("confirmation_token", "https://foo.us1.realm.cloud.io", new SyncUser.Callback<Void>() {
    @Override
    public void onSuccess(Void result) {
    }

    @Override
    public void onError(ObjectServerError error) {
    }
});
```
{% endtab %}

{% tab title="Javascript" %}
```typescript
Realm.Sync.User.confirmEmail('https://foo.us1.realm.cloud.io', 'confirmation_token').then(() => {
    // query sent successfully. 
}).catch((error) => {
    // an error has occurred
});
```
{% endtab %}

{% tab title=".NET" %}
```csharp
var serverUri = new Uri("https://foo.us1.realm.cloud.io");
await User.ConfirmEmailAsync(serverUri, "confirmation_token");
```
{% endtab %}
{% endtabs %}

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

