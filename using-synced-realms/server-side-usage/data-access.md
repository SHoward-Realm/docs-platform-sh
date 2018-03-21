---
description: Realm Platform allows you to access and change any shared Realm server-side.
---

# Data access

## Overview

Access to synchronized data is not limited to just mobile clients. Realm Platform offers support for server-side access via the following mechanisms:

* [Javascript \(Node.js\) SDK](https://realm.io/docs/javascript/latest/)
* [.Net SDK](https://realm.io/docs/dotnet/latest/)
* [GraphQL](../../graphql-web-access/)

You can use these mechanisms to query or write to the synchronized Realms, opening up possibilities for server-side integrations, logic, or pushing changes to mobile clients. In general, a good way to think about server-side functionality is as if it were a special "server-side client" participating in data sync just like mobile clients, but has trusted access to all application data.

### Admin User

Realm Platform's user authentication supports the concept of an admin user. These users automatically have full read/write access to all synchronized Realms. Typically, when building server-side logic you will use an admin user as the identity in the server-side client code. For more information on admin users see the guide in the user authentication section:

{% page-ref page="../working-with-users/admin-users.md" %}

## SDKs

Both Realm Javascript and Realm .Net support server-side environments. The API remains the same, allowing you to login with an admin user and then open any synchronized Realm.

{% tabs %}
{% tab title="Node.js" %}
Realm Javascript supports any 64-bit Linux running Glibc 2.12 or newer, such as Ubuntu, Fedora, Debian, or CentOS.

Simply install Realm via NPM:

```bash
npm install realm
```

Then create a script that logs in with an admin user and open any synchronized Realm:

```text
var Realm = require('realm');

const server_address = "INSERT SERVER ADDRESS";

async function main() {
    const adminUser = await Realm.Sync.User.login(`http://${server_address}`, 'admin', 'password');
    const realm = await Realm.open({
        sync: {
            user: adminUser,
            url: `realm://${server_address}/aRealm`,
        }
    });
    
    // Use the Realm for querying or writing to push data to clients
}
 
main()
```

For more information consult the [Realm Javascript SDK documentation](https://realm.io/docs/javascript/latest/).
{% endtab %}

{% tab title=".Net" %}
### Prerequisites

We support the following platforms:

* .NET Core 1.1 or later on:
  * Ubuntu 16.04 or later
  * Debian 8 or later
  * RHEL 7.1 or later
  * macOS 10.11 or later
  * Windows 10 or later

No explicit steps are required to enable Realm Platform. It is installed by default in the standard Realm NuGet.

Follow the installation instructions [here](https://realm.io/docs/dotnet/latest/#installation).

Then create a script that logs in with an admin user and open any synchronized Realm:

```csharp
var credentials = Credentials.UsernamePassword("admin", "password", createUser: false);
var authURL = new Uri("http://my.realm-server.com:9080");
var adminUser = await User.LoginAsync(credentials, authURL);

var serverURL = new Uri("realm://my.realm-server.com:9080/aRealm");
var configuration = new SyncConfiguration(adminUser, serverURL);

var realm = Realm.GetInstance(configuration);
```

For more information consult the [Realm .Net documentation](https://realm.io/docs/dotnet/latest/).
{% endtab %}
{% endtabs %}

## GraphQL

Realm Platform's GraphQL support is designed for use with both front-end Javascript applications and in server-side environments which lack a dedicated Realm SDK. For example, you could use the GraphQL API to query for data in server-side Java application or to write changes from a Python script. Given that GraphQL is just a query language for an underlying HTTP API, you can manually create the API calls or use [existing libraries.](http://graphql.org/code/)

For more information on Realm Platform's GraphQL support consult the full documentation:

{% page-ref page="../../graphql-web-access/" %}





