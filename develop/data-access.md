# Data Access

The Standard edition of the Realm Platform allow you to access and change any shared Realm server-side using an admin user.  

## Open a Realm with an Admin User

Assuming that we are running our Realm Object Server locally, we could use the following code.  

{% hint style="info" %}
Make sure to edit your server URL, feature token, and schema as necessary
{% endhint %}

Before you get started, make sure to install the `realm-js` package via NPM.  

```bash
npm install realm
```

Then create a new javascript file like the following:

{% code-tabs %}
{% code-tabs-item title="index.js" %}
```javascript
var fs = require('fs');
var Realm = require('realm');

//insert your feature token
var token = "eyJhbGciOi...";
// Unlock Standard Edition APIs
Realm.Sync.setFeatureToken(token); 

const server_url = '127.0.0.1:9080'; 

async function main() {
    //login as an admin user 
    adminUser = await Realm.Sync.User.login(`http://${server_url}`, 'admin', 'password')
    // Open a Realm using the admin user
    var realm = new Realm({
        sync: {
        user: adminUser,
        url: `realm://${server_url}/my-realm`,
        },
        schema: [{
            name: 'Person',
            properties: {
                name:     'string',
                birthday: 'date',
                picture:  'data?' // optional property
            }
        }]
    });
}

main()
```
{% endcode-tabs-item %}
{% endcode-tabs %}

You could then acess the data and view it in the console with the following: 

{% code-tabs %}
{% code-tabs-item title="index.js" %}
```javascript
var fs = require('fs');
var Realm = require('realm');

var token = "eyJhbGciOi...";
// Unlock Standard Edition APIs
Realm.Sync.setFeatureToken(token); 

const server_url = '127.0.0.1:9080'; 

async function main() {
    //login as an admin user 
    adminUser = await Realm.Sync.User.login(`http://${server_url}`, 'admin', 'password')
    // Open a Realm using the admin user
    // Open a Realm using the admin user
    var realm = new Realm({
        sync: {
        user: adminUser,
        url: `realm://${server_url}/my-realm`,
        }
    });
  
  var realmPersons = realm.objects('Person'); 
  console.log(realmPersons); 
}

main()
```
{% endcode-tabs-item %}
{% endcode-tabs %}

You can run the script above via the terminal like: 

```bash
node index.js
```

For more details on querying and working with the data, consult your specific language binding: 

* [JavaScript](https://realm.io/docs/javascript/latest/#queries) 
* [.Net](https://realm.io/docs/dotnet/latest/#queries)



Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

