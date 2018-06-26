# Anonymous Authentication

Sometimes it's useful to create a synchronized Realm for a user without prompting them for credentials - e.g. in an e-commerce application where any disruption leads to user churn. For such cases, the Realm Object Server exposes an anonymous provider that requires no payload and creates a new user for every request \(which means that you must use the built-in client-side user caching to avoid resetting the app state on every launch\).

By default the `AnonymousAuthProvider` is **enabled** when creating a project via `ros init`.

### Disable Anonymous Authentication

When creating a ROS project via `ros init` the `index` file that starts the server utilizes the `BasicServer` class. This wrapper provides default values, including enabling the `AnonymousAuthProvider`. To disable, you can set the authentication providers manually with, including only the ones needed:

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

### Enable Anonymous Authentication

While included by default, if you need to include the `AnonymousAuthProvider` manually, edit your ROS `index` file to pass in the `AnonymousAuthProvider` with the server configuration:

```javascript
const RealmObjectServer = require('realm-object-server');
const path = require('path');

const server = new RealmObjectServer.BasicServer();

const anonymousProvider = new RealmObjectServer.auth.AnonymousAuthProvider();

server.start({
    dataPath: path.join(__dirname, '../data'),
    authProviders: [ anonymousProvider ],
});
```



Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

