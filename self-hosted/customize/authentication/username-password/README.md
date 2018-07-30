# Username/Password

Realm Object Server provides a built-in username and password authentication provider. This is great to quickly get started with but isn’t designed to be a full-featured authentication system \(such as providing email capabilities\).

When you start Realm Object Server via:

```bash
ros start
```

The username/password provider is included and a default admin is created with the following credentials:

```text
username = 'realm-admin'
password = '' // Empty string
```

To customize the username/password provider, create a ROS project via `ros init`: \(then start the server with `npm start` to use the custom configuration\)

{% code-tabs %}
{% code-tabs-item title="src/index.ts" %}
```typescript
import { auth, BasicServer, FileConsoleLogger } from 'realm-object-server'
import * as path from 'path'

const server = new BasicServer()

// Customize username/password provider
let usernamePasswordProvider = new auth.PasswordAuthProvider({
    autoCreateAdminUser: true,
    saltLength: 32,
    iterations: 10000,
    keyLength: 512,
    digest: "sha512"
});

server.start({
  // Other options...
  authProviders: [ usernamePasswordProvider ],
}).catch((err) => {
  console.error("There was an error starting your custom ROS Server", err);
});
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3)

