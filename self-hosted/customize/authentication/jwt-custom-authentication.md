# JWT Custom Authentication

#### JWT Custom Authentication {#jwt-custom-authentication}

The JSON Web Token provider allows you to integrate with an existing authentication system. It requires that you modify the authentication server to expose a flow that produces signed JSON Web Tokens that your app then transmits to the Realm Object Server for verification.

**Issuing JWTs**

The only prerequisite is that your authentication server can issue RS256 signed JSON Web Tokens that contain a payload with a `userId` field that represents a unique user identifier - this will be the Id of the user in Realm Object Server. If you already have that in place, you can skip to the [enabling the JWT provider](#enabling-the-jwt-provider) section.

**Generating RS256 key**

To generate an RS256 key, you can use the following snippet:

```bash
openssl genrsa -des3 -out private.pem 4096
// Enter and confirm password when prompted
openssl rsa -in private.pem -outform PEM -pubout -out public.pem
```

The `private.pem` is the private key you'll use to sign the tokens, and `public.pem` is the public key you'll supply to Realm Object Server to verify them.

**Issuing tokens**

There are a variety of libraries that support generating and signing custom tokens. The code sample will use auth0's [node-jsonwebtoken](https://github.com/auth0/node-jsonwebtoken), but it can easily be translated to a language of choice. For a complete list, check out [jwt.io](https://jwt.io).

To issue a custom token, add `jsonwebtoken` to your project by running `npm install jsonwebtoken`. Then copy `private.pem` generated in the previous step to a well known location and execute the following js code:

```typescript
const jwt = require('jsonwebtoken');
const fs = require('fs');
const key = fs.readFileSync('private.pem');

const payload = {
  userId: '123',
  isAdmin: true // optional
  // other properties (ignored by Realm Object Server)
};

const token = jwt.sign(payload, { key:  key, passphrase: 'your-passphrase' }, { algorithm: 'RS256'});
// Send token to your client app
```

The `isAdmin` field in the payload is optional and if it is set to `true`, Realm Object Server will authenticate the user as admin user. If not set or set to `false`, the user will be a regular user. You can add additional properties in the payload but they'll be ignored by Realm Object Server.

**Enabling the JWT provider**

Let's assume we have a publicKey that looks like: 

{% code-tabs %}
{% code-tabs-item title="key.pub" %}
```text
-----BEGIN PUBLIC KEY-----
MIICIjANBgkqhki...
-----END PUBLIC KEY-----
```
{% endcode-tabs-item %}
{% endcode-tabs %}

To include and customize the JWT provider, create a Realm Object Server project via `ros init`:

```javascript
const RealmObjectServer = require('realm-object-server');
const path = require('path');
const auth = RealmObjectServer.auth;

const server = new RealmObjectServer.BasicServer();

// Add your public key from "Generating RS256 key" section
let jwtProvider = new auth.JwtAuthProvider(
  {
    publicKey: '-----BEGIN PUBLIC KEY-----\nMIICIjANBgkqhki...\n-----END PUBLIC KEY-----'
  }
)

server.start({
    dataPath: path.join(__dirname, '../data'),
    authProviders: [ jwtProvider ],
}).catch((err) => {
    console.error("There was an error starting your custom Realm Object Server", err);
});
```

{% hint style="info" %}
It is safest to copy and paste your public key into the index file.  Use `\n` to preserve newlines within your token.  
{% endhint %}

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

