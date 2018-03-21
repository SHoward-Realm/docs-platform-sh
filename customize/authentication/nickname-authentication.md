# Nickname Authentication

The Nickname provider allows you to authenticate users just by their nickname without requiring passwords or other identifying information. This is useful when prototyping collaborative features for your app without worrying about the actual login flow until later in the development lifecycle.

{% hint style="danger" %}
The Nickname provider is not secure and should never be enabled in production deployments. It's meant to only be used during development.
{% endhint %}

By default the Nickname provider is **enabled** when creating a project via `ros init`.

### Disable Nickname Authentication

When creating a ROS project via `ros init` the `index` file that starts the server utilizes the `BasicServer` class. This wrapper provides default values, including enabling the `NicknameAuthProvider` . Once you are ready to go into production, you should disable the `NicknameAuthProvider` by removing it from the provider list:

```javascript
const RealmObjectServer = require('realm-object-server');
const path = require('path');

const server = new RealmObjectServer.BasicServer();

// Create the necessary authentication providers needed
const passwordProvider = new PasswordAuthProvider({ autoCreateAdminUser: true });

server.start({
    dataPath: path.join(__dirname, '../data'),
    
    // Set the providers manually to override defaults
    authProviders: [ passwordProvider ],
});
```

### Enable Nickname Authentication

While included by default, if you need to include the `NicknameAuthProvider` manually, edit your ROS `index` file to pass in the `NicknameAuthProvider` with the server configuration:

```javascript
const RealmObjectServer = require('realm-object-server');
const path = require('path');
const server = new RealmObjectServer.BasicServer();

const nicknameProvider = new NicknameAuthProvider();

server.start({
    dataPath: path.join(__dirname, '../data'),
    authProviders: [ nicknameProvider ],
});
```





Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

