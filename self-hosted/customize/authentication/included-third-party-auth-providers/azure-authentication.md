# Azure Authentication

The Realm Object Server package includes a pre-built provider for Azure Active Directory. Clients authenticate through Azure SDK/API and sends the access token to ROS. This provider handles validating the JWT access token and then authenticating in Realm.

To include or customize the Azure provider, create a ROS project via `ros init`:

```typescript
import { AzureAuthProvider } from '../node_modules/realm-object-server/dist/auth/azure/AzureAuthProvider'

const RealmObjectServer = require('realm-object-server');
const path = require('path');

const server = new RealmObjectServer.BasicServer();

// Add your Directory ID from the Azure Portal (under "Properties")
let azureProvider = new AzureAuthProvider(
  {
    tenant_id: '81560d038272f7ffae5724220b9e9ea75d6e3f18'
  }
)

server.start({
    dataPath: path.join(__dirname, '../data'),
    authProviders: [ azureProvider ],
}).catch((err) => {
    console.error("There was an error starting your custom ROS Server", err);
});
```





Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

