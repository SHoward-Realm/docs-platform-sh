# Google Authentication

The Realm Object Server package includes a pre-built provider for Google. Clients authenticate through Google SDK/API and sends the access token to ROS. This provider handles validating the access token with Google’s API, and then authenticating in Realm. To setup, you must obtain a client Id described in this guide: [https://developers.google.com/identity/protocols/OAuth2InstalledApp](https://developers.google.com/identity/protocols/OAuth2InstalledApp)

To include the Google provider, create a ROS project via `ros init`:

```text
const RealmObjectServer = require('realm-object-server');
const path = require('path');

const server = new RealmObjectServer.BasicServer();
const googleProvider = new GoogleAuthProvider({
    clientId: '012345678901-abcdefghijklmnopqrstvuvwxyz01234.apps.googleusercontent.com'
})

server.start({
    dataPath: path.join(__dirname, '../data'),
    authProviders: [ googleProvider ],
}).catch((err) => {
    console.error("There was an error starting your custom ROS Server", err);
});
```





Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

