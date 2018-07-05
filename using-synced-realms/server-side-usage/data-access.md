# Data access

## Overview

Access to synchronized data is not limited to just mobile clients. Realm Platform offers support for server-side access via the following mechanisms:

* [Javascript \(Node.js\) SDK](https://realm.io/docs/javascript/latest/)
* [.Net SDK](https://realm.io/docs/dotnet/latest/)
* [GraphQL](../../graphql-web-access/)

You can use these mechanisms to query or write to the synchronized Realms, opening up possibilities for server-side integrations, logic, or pushing changes to mobile clients. In general, a good way to think about server-side functionality is as if it were a special "server-side client" participating in data sync just like mobile clients, but has trusted access to all application data.

### Admin User

Realm Platform's user authentication supports the concept of an admin user. These users automatically have full read/write access to all synchronized Realms. Typically, when building server-side logic you will use an admin user as the identity in the server-side client code. For more information on admin users see the guide in the user authentication section:

{% page-ref page="../user-authentication/admin-users.md" %}

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

```javascript
var Realm = require('realm');

const server_address = "INSERT SERVER ADDRESS";

async function main() {
    const adminUser = await Realm.Sync.User.login(`https://${server_address}`, 'admin', 'password');
    const realm = await Realm.open({
        sync: {
            user: adminUser,
            url: `realms://${server_address}/aRealm`,
        }
    });

    // Use the Realm for querying or writing to push data to clients
}
 
main()
```

For more information consult the [Realm Javascript SDK documentation](https://realm.io/docs/javascript/latest/).
{% endtab %}

{% tab title=".Net" %}
Realm .NET supports running .NET Core and .NET Framework apps on Windows Server 2012 or later, as well as all Linux distributions that .NET Core supports.

Simply install Realm via [NuGet](https://www.nuget.org/packages/Realm/):

```text
Install-Package Realm
```

**Note**: The `dotnet` command line tool is not supported when building for .NET Core. Please use Visual Studio or the `msbuild` command line tool to build your project.

Then create a file called `FodyWeavers.xml` in the root of your project and populate it with the following content:

```markup
<?xml version="1.0" encoding="utf-8" ?>
<Weavers>
    <RealmWeaver />
</Weavers>
```

Finally, log in as an admin user and open any synchronized Realm:

```csharp
const string ServerUrl = "MY-SERVER-URL"
async Task UseRealm()
{
    var credentials = Credentials.UsernamePassword("admin", "password", createUser: false);
    var adminUser = await User.LoginAsync(credentials, new Uri($"https://{ServerUrl}"));
    var configuration = new FullSyncConfiguration(new Uri("/aRealm", UriKind.Relative), adminUser)
    {
        IsDynamic = true
    };
    var realm = await Realm.GetInstanceAsync(configuration);

    // Use the Realm for querying or writing to push data to clients
}

static void Main()
{
    // Make sure to use a thread with a synchronization context in order
    // to ensure that task continuations will be dispatched on the same
    // thread. In this example, we're using AsyncContext Nito.AsyncEx
    // package: https://www.nuget.org/packages/Nito.AsyncEx
    AsyncContext.Run(() => UseRealm());
}
```

For more information consult the [Realm .NET SDK documentation](http://realm.io/docs/dotnet/latest/).
{% endtab %}
{% endtabs %}

## GraphQL

Realm Platform's GraphQL support is designed for use with both front-end Javascript applications and in server-side environments which lack a dedicated Realm SDK. For example, you could use the GraphQL API to query for data in server-side Java application or to write changes from a Python script. Given that GraphQL is just a query language for an underlying HTTP API, you can manually create the API calls or use [existing libraries.](http://graphql.org/code/)

For more information on Realm Platform's GraphQL support consult the full documentation:

{% page-ref page="../../graphql-web-access/" %}

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3)

