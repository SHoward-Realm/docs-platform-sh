# Facebook Authentication

The Realm Object Server package includes a pre-built provider for Facebook. Clients authenticate through Facebook SDK/API and sends the access token to ROS. This provider handles validating the access token with Facebook’s API, and then authenticating in Realm.

To include the Facebook provider, create a ROS project via `ros init`:

```text
import { auth } from 'realm-object-server';

const facebookProvider = new auth.FacebookAuthProvider()

const RealmObjectServer = require('realm-object-server');
const path = require('path');

const server = new RealmObjectServer.BasicServer();
const facebookProvider = new FacebookAuthProvider()

server.start({
    dataPath: path.join(__dirname, '../data'),
    authProviders: [ facebookProvider ],
}).catch((err) => {
    console.error("There was an error starting your custom ROS Server", err);
});
```





Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

