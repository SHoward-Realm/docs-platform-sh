# Data change events

Realm Platform offers the ability to register for data change events, also called "Event Handling." This functionality is provided in the [server-side Node.js and .NET SDKs](data-access.md#overview) via a global event listener API which hooks into the Realm Object Server, allowing you to observe changes across Realms. This could mean listening to every Realm for changes, or Realms that match a specific pattern. For example, if your app architecture separated user settings data into a Realm unique for each user where the virtual path was `/~/settings`, then a listener could be setup to react to changes to any user’s `settings` Realm.

Whenever a change is synchronized to the server, it triggers a notification which allows you to run custom server-side logic in response to the change. The notification will inform you about the virtual path of the updated Realm and provide the Realm object and fine-grained information on which objects changed. The change set provides the object indexes broken down by class name for any inserted, deleted, or modified object in the last synchronized transaction.

## Creating Event Handlers in Node.js

To use Realm's event handling, you’ll need to create a small Node.js application.

Create a directory to place the server files, then create a file named `package.json`. This JSON file is used by Node.js and npm, its package manager, to describe an application and specify external dependencies.

You can create this file interactively by using `npm init`. You can also fill in a simple skeleton file yourself using your text editor:

```javascript
{
  "name": "MyApp",
  "version": "0.0.1",
  "main": "index.js",
  "author": "Your Name",
  "description": "My Cool Realm App",
  "dependencies":
    {
      "realm":"latest"
    }
}
```

After the `package.json` file is configured properly, type:

```bash
npm install
```

to download, unpack and configure all the modules and their dependencies.

Your event handler will need to access the Object Server with administrative privileges. In this example, we use the default `realm-admin` and specify our credentials in our `Realm.Sync.User.login` call.

This can be used to synchronously construct a`Realm.Sync.User`object which can be passed into the`Realm`constructor to open a connection to any Realm on the server side.

A sample `index.js` file might look something like this. This example listens for changes to a user-specific private Realm at the virtual path `/~/private`. It will look for updated `Coupon` objects in these Realms, verify their coupon code if it wasn’t verified yet, and write the result of the verification into the `isValid` property of the `Coupon` object.

```javascript
'use strict';

var Realm = require('realm'); 

// the URL to the Realm Object Server
var SERVER_URL = '//127.0.0.1:9080';

// The regular expression you provide restricts the observed Realm files to only the subset you
// are actually interested in. This is done in a separate step to avoid the cost
// of computing the fine-grained change set if it's not necessary.
var NOTIFIER_PATH = '^/([^/]+)/private$';

//declare admin user 
let adminUser = undefined

// The handleChange callback is called for every observed Realm file whenever it
// has changes. It is called with a change event which contains the path, the Realm,
// a version of the Realm from before the change, and indexes indication all objects
// which were added, deleted, or modified in this change
var handleChange = async function (changeEvent) {
  // Extract the user ID from the virtual path, assuming that we're using
  // a filter which only subscribes us to updates of user-scoped Realms.
  var matches = changeEvent.path.match("^/([^/]+)/([^/]+)$");
  var userId = matches[1];

  var realm = changeEvent.realm;
  var coupons = realm.objects('Coupon');
  var couponIndexes = changeEvent.changes.Coupon.insertions;

  for (let couponIndex of couponIndexes) {
    var coupon = coupons[couponIndex];
    if (coupon.isValid !== undefined) {
      var isValid = verifyCouponForUser(coupon, userId);
      // Attention: Writes here will trigger a subsequent notification.
      // Take care that this doesn't cause infinite changes!
      realm.write(function() {
        coupon.isValid = isValid;
      });
    }
  }
}

function verifyCouponForUser(coupon, userId) {
    //logic for verifying a coupon's validity
}

// register the event handler callback
async function main() {
    adminUser = await Realm.Sync.User.login(`https:${SERVER_URL}`, 'realm-admin', '')
    Realm.Sync.addListener(`realms:${SERVER_URL}`, adminUser, NOTIFIER_PATH, 'change', handleChange);
}

main()
```

The heart of the event handler is the`handleChange()`function, which is passed a`changeEvent`object. This object has four keys:

* `path`: The path of the changed Realm \(used above with `match` to extract the user ID\)
* `realm`: the changed Realm itself
* `oldRealm`: the changed Realm in its _old_ state, before the changes were applied
* `changes`: an object containing a hash map of the Realm’s changed objects

The `changes` object itself has a more complicated structure: it’s a series of key/value pairs, the keys of which are the names of objects \(i.e., `Coupon` in the above code\), and the values of which are _another_ object with key/value pairs listing insertions, deletions, and modifications to those objects. The values of those keys are index values into the Realm. Here’s the overall structure of the`changeEvent`object:

```javascript
{
  path: "realms://server/user/realm",
  realm: <realm object>,
  oldRealm: <realm object>,
  changes: {
    objectType1: {
      insertions: [ a, b, c, ...],
      deletions: [ a, b, c, ...],
      modifications: [ a, b, c, ...]
    },
    objectType2: {
      insertions: [ a, b, c, ...],
      deletions: [ a, b, c, ...],
      modifications: [ a, b, c, ...]
    }
  }
}
```

In the example above, we get the Coupons and the indexes of the newly inserted coupons with this:

```javascript
var realm = changeEvent.realm;
var coupons = realm.objects('Coupon');
var couponIndexes = changeEvent.changes.Coupon.insertions;
```

Then, we use `for (let couponIndex of couponIndexes)` to loop through the indexes and to get each changed coupon.

## Creating Event Handlers in .NET

To use Realm Event Handling, you’ll need to create a .NET application. It can be a console app or Asp.NET app and can run on all flavours of Linux, macOS, or Windows that the [.NET SDK supports](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/docs/dotnet/latest/README.md).

Create a new project or open your existing one and add the [Realm.Server](https://www.nuget.org/packages/Realm.Server) NuGet package.

You’ll need to create a class that implements the [INotificationHandler](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.INotificationHandler.html) interface. It has two methods - [ShouldHandle](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.INotificationHandler.html) and [HandleChangesAsync](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.INotificationHandler.html).

To tell the notifier that your handler is interested in observing changes for a particular path, return `true` in the `ShouldHandle` callback. As a general principle, `ShouldHandle` should always return stable result every time it is invoked with the same path and should return as quickly as possible to avoid blocking observing of notifications. We’ve provided an abstract implementation of the `INotificationHandler` interface in a [Regex​Notification​Handler](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.RegexNotificationHandler.html) class that implements `ShouldHandle` in terms of regex matching the string that is passed to its constructor. For example, if you want to observe changes to the user-specific private Realm at virtual path `/~/private`, you would provide the following implementation:

```csharp
class PrivateHandler : RegexNotificationHandler
{
    public PrivateHandler() : base("^/.*/private$")
    {
    }

    public override async Task HandleChangeAsync(IChangeDetails details)
    {
        // TODO: handle change notifications
    }
}
```

The `HandleChangesAsync` method is the heart of the event handler - it gets invoked whenever a Realm at an observed path \(`ShouldHandle` returned `true`\) changes and is passed an [IChangeDetails](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.IChangeDetails.html) object that contains detailed information about the change that occurred. It has the following properties and methods:

*  [PreviousRealm](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.IChangeDetails.html) and [CurrentRealm](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.IChangeDetails.html) are readonly snapshots of the state of the Realm just before and just after the change has occurred. `PreviousRealm` may be null if the notification was received just after creating the Realm. `CurrentRealm` can never be null.
*  [RealmPath](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.IChangeDetails.html) is the virtual path of the Realm that has changed, e.g. `/some-user-id/private`.
*  [GetRealmForWriting\(\)](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.IChangeDetails.html) can be invoked to get a writeable instance of the Realm that has changed in case you need to write some data in response to the change notification. Since writing to any Realm automatically advances it to the latest version, this Realm instance may contain slightly newer data than `CurrentRealm` if new changes have been received while handling the notification. Unlike `CurrentRealm` and `PreviousRealm`, the lifetime of this instance is not managed by the notifier, so make sure to place it inside a `using` statement or manually dispose it as soon as you’re done with it.
*  [Changes](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.IChangeDetails.html) is a dictionary of class names and [IChangeSetDetails](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.IChangeSetDetails.html) objects. For each object type that has changed, you’ll get a corresponding `IChangeSetDetails` object containing [Insertions](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.IChangeSetDetails.html), [Deletions](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.IChangeSetDetails.html), and [Modifications](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.IChangeSetDetails.html) collections. Each of those contains [IModification​Details](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.IModificationDetails.html) instances with the following properties:
  *  [PreviousIndex](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.IModificationDetails.html) indicates the index of the changed object in the `PreviousRealm` view. It will be `-1` if the object was inserted.
  *  [PreviousObject](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.IModificationDetails.html) is the state of the object just before it has changed. It will be `null` if the object was inserted.
  *  [CurrentIndex](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.IModificationDetails.html) indicates the index of the changed object in the `CurrentRealm` view. It will be `-1` if the object was deleted.
  *  [CurrentObject](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.IModificationDetails.html) is the state of the object just after it has changed. It will be `null` if the object was deleted.

Here’s what a sample event handling application might look like in .NET:

```csharp
public class Program
{
    private const string AdminUsername = "admin@foo.com";
    private const string AdminPassword = "super-secure-password";
    private const string ServerUrl = "127.0.0.1:9080";

    public static void Main(string[] args) => MainAsync().Wait();

    public static async Task MainAsync()
    {

        // Login the admin user
        var credentials = Credentials.UsernamePassword(AdminUsername, AdminPassword, createUser: false);
        var admin = await User.LoginAsync(credentials, new Uri($"http://{ServerUrl}"));

        var config = new NotifierConfiguration(admin)
        {
            // Add all handlers that this notifier will invoke
            Handlers = { new CouponHandler() }
        };

        // Start the notifier. Your handlers will be invoked for as
        // long as the notifier is not disposed.
        using (var notifier = await Notifier.StartAsync(config))
        {
            do
            {
                Console.WriteLine("Type in 'exit' to quit the app.");
            }
            while (Console.ReadLine() != "exit");
        }
    }

    class CouponHandler : RegexNotificationHandler
    {
        // The regular expression you provide restricts the observed Realm files
        // to only the subset you are actually interested in. This is done to 
        // avoid the cost of computing the fine-grained change set if it's not
        // necessary.
        public CouponHandler() : base($"^/.*/private$")
        {
        }

        // The HandleChangeAsync method is called for every observed Realm file 
        // whenever it has changes. It is called with a change event which contains 
        // a version of the Realm from before and after the change, as well as
        // collections of all objects which were added, deleted, or modified in this change
        public override async Task HandleChangeAsync(IChangeDetails details)
        {
            if (details.TryGetValue("Coupon", out var changeSetDetails) &&
                changeSetDetails.Insertions.Length > 0)
            {
                // Extract the user ID from the virtual path, assuming that we're using
                // a filter which only subscribes us to updates of user-scoped Realms.
                var userId = details.RealmPath.Split(new[] { '/' }, StringSplitOptions.RemoveEmptyEntries)[0];

                using (var realm = details.GetRealmForWriting())
                {
                    foreach (var coupon in changeSetDetails.Insertions.Select(c => c.CurrentObject))
                    {
                        var isValid = await CouponVerifier.VerifyAsync(coupon, userId);

                        // Create a ThreadSafeReference of the coupon. While both
                        // details.CurrentRealm and details.GetRealmForWriting() are open 
                        // on the same thread, they are at different versions, so you need
                        // to pass the Coupon between them either via ThreadSafeReference
                        // or by its PrimaryKey.
                        var writeableCoupon = realm.ResolveReference(ThreadSafeReference.Create(coupon));

                        // It may be null if the coupon was deleted by the time we get here
                        if (writeableCoupon != null)
                        {
                            realm.Write(() => writeableCoupon.IsValid = isValid);
                        }
                    }
                }
            }
        }
    }
}
```

**Notes**

* Multiple Notifiers may be started \(via [Notifier.StartAsync](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.Notifier.html)\) but they need to have a different [WorkingDirectory](https://github.com/realm/realm-docs/tree/31514dd9fad29d848cb993e184bc8746e80cee59/dotnet/latest/api/reference/Realms.Server.NotifierConfiguration.html) specified to avoid errors.
* `HandleChangesAsync` will be invoked in parallel for different Realms but sequentially for changes on a single Realm. This means that if your code takes a lot of time to return from the function, a queue of notifications may build up for that particular Realm. Make sure to design your architecture with that in mind.
* Asynchronous calls inside `HandleChangesAsync` **must not** use `ConfigureAwait(false)` as that will dispatch the continuation on a different thread making all Realm and RealmObject instances \(that were open prior to the async operation\) inaccessible from that thread.

## Handling Changes {#integrating-with-a-3rd-party-api}

The event handler callback provides access to detailed change information through a passed in change event object. This includes the indexes for the objects corresponding to:

* Insertions
* Modifications
* Deletions

The change information only applies at an object-level. If you need property-level change information, an additional data adapter API is available in Javascript which is designed to pass every database operation. It forms the basis for our pre-built database connectors. Read more here:

{% page-ref page="data-integration/" %}

### Insertions/Modifications

To access the inserted or modified objects, you can access the `Realm` object included in the change event object:

{% tabs %}
{% tab title="Javascript" %}
```javascript
var handleChange = async function (changeEvent) {
  // Get the current Realm
  var realm = changeEvent.realm;
  // Retrieve all objects of the relevant type
  var coupons = realm.objects('Coupon');
  // Retrieve the indexes for the insertions/modifications
  var couponIndexes = changeEvent.changes.Coupon.insertions;

  for (let couponIndex of couponIndexes) {
    // Use the Results object to retrieve the inserted/modified object
    var coupon = coupons[couponIndex];

    //..
  }
}
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// The HandleChangeAsync method is called for every observed Realm file 
// whenever it has changes. It is called with a change event which contains 
// a version of the Realm from before and after the change, as well as
// collections of all objects which were added, deleted, or modified in this change
public override async Task HandleChangeAsync(IChangeDetails details)
{
    // Retrieve the indexes for the insertions/modifications
    if (details.TryGetValue("Coupon", out var changeSetDetails) &&
        changeSetDetails.Insertions.Length > 0)
    {
        // Get the current Realm
        // If you want a read-only version, use .CurrentRealm property
        using (var realm = details.GetRealmForWriting())
        {
            // Use the Results object to retrieve the inserted/modified object
            foreach (var coupon in changeSetDetails.Insertions.Select(c => c.CurrentObject))
            {
                //..
            }
        }
    }
}
```
{% endtab %}
{% endtabs %}

### Deletions

To access the deleted objects, you cannot use the `Realm` object included in the change event object. The reason is that the `Realm`object is at the current state of the database after the changes have been applied. This means the deleted objects are already removed. However, because Realm has an MVCC architecture, it is possible to provide a second view of the database that is at the state before the change.

This previous or old state is provided in the `oldRealm` object included in the change event object. Use this to retrieve the delete objects:

{% tabs %}
{% tab title="Javascript" %}
```javascript
var handleChange = async function (changeEvent) {
  // Get the old Realm that is at the state before the change
  var oldRealm = changeEvent.oldRealm;
  // Retrieve all objects of the relevant type
  var coupons = realm.objects('Coupon');
  // Retrieve the indexes for the deletions
  var couponIndexes = changeEvent.changes.Coupon.deletions;

  for (let couponIndex of couponIndexes) {
    // Use the Results object to retrieve the deleted object
    var deletedCoupone = coupons[couponIndex];

    //..
  }
}
```
{% endtab %}

{% tab title=".Net" %}
```csharp
// The HandleChangeAsync method is called for every observed Realm file 
// whenever it has changes. It is called with a change event which contains 
// a version of the Realm from before and after the change, as well as
// collections of all objects which were added, deleted, or modified in this change
public override async Task HandleChangeAsync(IChangeDetails details)
{
    // Retrieve the indexes for the insertions/modifications
    if (details.TryGetValue("Coupon", out var changeSetDetails) &&
        changeSetDetails.Deletions.Length > 0)
    {
        // Get the previous Realm
        // This Realm is read-only!
        using (var realm = details.PreviousRealm)
        {
            // Use the Results object to retrieve the deleted object
            foreach (var coupon in changeSetDetails.Deletions.Select(c => c.CurrentObject))
            {
                //..
            }
        }
    }
}
```
{% endtab %}
{% endtabs %}

## Integrating with a 3rd Party API {#integrating-with-a-3rd-party-api}

A great potential use case for our event handler is integration with a 3rd party API. When a change is made to the Realm Object Server, a call can be made to a 3rd party API and then the results can easily be written to a synchronized Realm.

The following example was made to be easily used with our ToDo app tutorial. It takes advantage of the [wit.ai](https://wit.ai/) API which can be used for basic text recognition.

### Prerequisites {#prerequisites}

Before running the event handler, install the dependencies via NPM

```bash
npm install realmnpm install node-wit
```

If you choose to use this same API, you'll need to sign up to get an API token, and you'll need to configure a `wit/datetime` entity.

### Example {#example}

You'll need to fill out a number of constants within the script:

* Wit Access Token
* Server URL
* Admin Login Credentials \(this assumes the default realm-admin user and password\)

The script assumes communication over https

```javascript
'use strict';

var fs = require('fs');
var Realm = require('realm');
const { Wit, log } = require('node-wit');

// Server access token from wit.ai API
var WIT_ACCESS_TOKEN = "INSERT_YOUR_WIT_ACCESS_TOKEN";

// The URL to the Realm Object Server
//format should be: 'IP_ADDRESS:port'  like example below 
// var SERVER_URL = '127.0.0.1:9080';
var SERVER_URL = 'INSERT_SERVER_URL';

// The path used by the global notifier to listen for changes across all
// Realms that match.
var NOTIFIER_PATH = "/ToDo";

const client = new Wit({ accessToken: WIT_ACCESS_TOKEN })

//enable debugging if needed 
//Realm.Sync.setLogLevel('debug');

let adminUser = undefined
    //INSERT HANDLER HERE
var handleChange = async function(changeEvent) {
    const realm = changeEvent.realm;
    const tasks = realm.objects('Item');
    console.log(tasks);

    console.log('received change event');

    const taskInsertIndexes = changeEvent.changes.Item.insertions;
    const taskModIndexes = changeEvent.changes.Item.modifications;
    const taskDeleteIndexes = changeEvent.changes.Item.deletions;

    for (var i = 0; i < taskInsertIndexes.length; i++) {
        const task = tasks[taskInsertIndexes[i]];
        console.log(task);
        let itemId = task.itemId
        if (task !== undefined) {
            console.log('insertion occurred');
            const client = new Wit({ accessToken: WIT_ACCESS_TOKEN });
            const data = await client.message(task.body, {})
            console.log("Response received from wit: " + JSON.stringify(data));
            if (data.entities.datetime) {
                var dateTime = data.entities.datetime[0];
                if (!dateTime) {
                    console.log("Couldn't find a date.");
                    return;
                }
                realm.write(() => {
                    task.body = `${task.body} - Date: ${dateTime.value}`
                })
            }
        }
    }
}
async function main() {
    adminUser = await Realm.Sync.User.login(`https://${SERVER_URL}`, 'realm-admin', 'password')
    Realm.Sync.addListener(`realms://${SERVER_URL}`, adminUser, NOTIFIER_PATH, 'change', handleChange);
    console.log('listening');
}

main()
```

After filling out the constants, you can run the handler with node like:

```bash
node eventHandler.js
```

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3)

