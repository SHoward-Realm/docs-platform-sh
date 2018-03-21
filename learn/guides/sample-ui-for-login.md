# Sample UI for Login

The LoginKit can be pulled into your app to prompt the user for credentials and then use those credentials to open a synced Realm. User credentials are required to open a synced realm although they could be hardcoded into your app if your app does not require users to log in.

## Example Code {#example-code}

{% tabs %}
{% tab title="Swift" %}
```swift
// Create the object
let loginController = LoginViewController(style: .lightTranslucent) // init() also defaults to lightTranslucent

// Configure any of the inputs before presenting it
loginController.serverURL = "localhost"

// Set a closure that will be called on successful login
loginController.loginSuccessfulHandler = { user in
	// Provides the successfully authenticated SyncUser object
}
```
{% endtab %}

{% tab title="Objective-C" %}
```objectivec
// Create the object
RLMLoginViewController *loginController = [[RLMLoginViewController alloc] initWithStyle:LoginViewControllerStyleLightTranslucent];

// Configure any of the inputs before presenting it
loginController.serverURL = @"localhost";

// Set a closure that will be called on successful login
loginController.loginSuccessfulHandler = ^(RLMSyncUser *user) {
	// Provides the successfully authenticated RLMSyncUser object
};
```
{% endtab %}
{% endtabs %}

## Features {#features}

* Light & dark themes for light apps like Realm Draw, and dark apps like Realm Tasks.
* Fully adaptive to both smartphone, and tablet screen sizes.
* Easy swapping between 'log in' and 'sign up' modes.
* Optional settings to hide the server URL and 'remember me' form fields.
* The ability to remember username and passwords on subsequent app launches.

#### [Get Realm LoginKit](https://github.com/realm-demos/realm-loginkit/tree/master/RealmLoginKit%20Apple/) {#get-realm-loginkit}

* Xcode 8.0 and up
* iOS 9.0 and up

####  {#objective-c}



Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

