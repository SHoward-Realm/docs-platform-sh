# Convert a Local Realm to a Synced Realm

As your application grows to take advantage of more Realm features, you may add synchronization backed by the Realm Object Server---and this may require converting a locally-stored Realm file to a synchronized Realm. While the Realm Platform doesn't offer an automatic solution to this, it doesn't require a lot of code to accomplish. Theses example are for small applications to perform this task; the code could be easily integrated into an existing application.

{% hint style="info" %}
Currently, Javascript is the recommended language to perform this migration. We aim to have more code samples available in the future. If you are experiencing issues, please submit a ticket [here](https://support.realm.io/).
{% endhint %}

{% tabs %}
{% tab title="Objective-C" %}
{% code-tabs %}
{% code-tabs-item title="AppDelegate.h" %}
```objectivec
#import <UIKit/UIKit.h>

@interface AppDelegate : UIResponder <UIApplicationDelegate>

@property (strong, nonatomic) UIWindow *window;

@end
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% code-tabs %}
{% code-tabs-item title="AppDelegate.m" %}
```objectivec
#import "AppDelegate.h"

@import Realm;
@import Realm.Dynamic;
@import Realm.Private;

@interface AppDelegate ()

@end

@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    NSString *sourceFilePath = [[NSBundle mainBundle] pathForResource:@"fieldFlow" ofType:@"realm"];

    RLMRealmConfiguration *configuration = [[RLMRealmConfiguration alloc] init];
    configuration.fileURL = [NSURL URLWithString:sourceFilePath];
    configuration.dynamic = true;
    configuration.readOnly = YES;

    RLMRealm *localRealm = [RLMRealm realmWithConfiguration:configuration error:nil];

    RLMSyncCredentials *creds = [RLMSyncCredentials credentialsWithUsername:@"admin@realm.io" password:@"password" register:NO];
    [RLMSyncUser logInWithCredentials:creds authServerURL:[NSURL URLWithString:@"http://localhost:9080"] onCompletion:^(RLMSyncUser *syncUser, NSError *error) {
        dispatch_async(dispatch_get_main_queue(), ^{
            [self copyToSyncRealmWithRealm: localRealm user:syncUser];
        });
    }];

    return YES;
}

- (void)copyToSyncRealmWithRealm:(RLMRealm *)realm user:(RLMSyncUser *)user
{
    RLMRealmConfiguration *syncConfig = [[RLMRealmConfiguration alloc] init];
    syncConfig.syncConfiguration = [[RLMSyncConfiguration alloc] initWithUser:user realmURL:[NSURL URLWithString:@"realm://localhost:9080/~/fieldRow"]];
    syncConfig.customSchema = [realm.schema copy];

    RLMRealm *syncRealm = [RLMRealm realmWithConfiguration:syncConfig error:nil];
    syncRealm.schema = syncConfig.customSchema;

    [syncRealm transactionWithBlock:^{
        NSArray *objectSchema = syncConfig.customSchema.objectSchema;
        for (RLMObjectSchema *schema in objectSchema) {
            RLMResults *allObjects = [realm allObjects:schema.className];
            for (RLMObject *object in allObjects) {
                RLMCreateObjectInRealmWithValue(syncRealm, schema.className, object, true);
            }
        }
    }];
}

@end
```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endtab %}

{% tab title="Javascript" %}
```javascript
// Copy local realm to ROS

const Realm = require('realm');

// UPDATE THESE
const realm_server = 'localhost:9080';
const username     = 'myuser@realm.io'; // this is the user doing the copy
const password     = 'mypassword';
const source_realm_path = './myLocalRealm.realm'; // path on disk
const target_realm_path = '/~/myRemoteRealm'; // path on server

var copyObject = function(obj, objSchema, targetRealm) {
    const copy = {};
    for (var key in objSchema.properties) {
        const prop = objSchema.properties[key];
        if (!prop.hasOwnProperty('objectType')) {
            copy[key] = obj[key];
        }
        else if (prop['type'] == "list") {
            copy[key] = [];
        }
        else {
            copy[key] = null;
        }
    }

    // Add this object to the target realm
    targetRealm.create(objSchema.name, copy);
}

var getMatchingObjectInOtherRealm = function(sourceObj, source_realm, target_realm, class_name) {
    const allObjects = source_realm.objects(class_name);
    const ndx = allObjects.indexOf(sourceObj);

    // Get object on same position in target realm
    return target_realm.objects(class_name)[ndx];
}

var addLinksToObject = function(sourceObj, targetObj, objSchema, source_realm, target_realm) {
    for (var key in objSchema.properties) {
        const prop = objSchema.properties[key];
        if (prop.hasOwnProperty('objectType')) {
            if (prop['type'] == "list") {
                var targetList = targetObj[key];
                sourceObj[key].forEach((linkedObj) => {
                    const obj = getMatchingObjectInOtherRealm(linkedObj, source_realm, target_realm, prop.objectType);
                    targetList.push(obj);
                });
            }
            else {
                // Find the position of the linked object
                const linkedObj = sourceObj[key];
                if (linkedObj === null) {
                    continue;
                }

                // Set link to object on same position in target realm
                targetObj[key] = getMatchingObjectInOtherRealm(linkedObj, source_realm, target_realm, prop.objectType);
            }
        }
    }
}

var copyRealm = function(user, local_realm_path, remote_realm_url) {
    // Open the local realm
    const source_realm =  new Realm({path: local_realm_path});
    const source_realm_schema = source_realm.schema;

    // Create the new realm (with same schema as the source)
    const target_realm = new Realm({
        sync: {
            user: user,
            url:  remote_realm_url,
        },
        schema: source_realm_schema
    });

    target_realm.write(() => {
        // Copy all objects but ignore links for now
        source_realm_schema.forEach((objSchema) => {
            console.log("copying objects:", objSchema['name']);
            const allObjects = source_realm.objects(objSchema['name']);

            allObjects.forEach((obj) => {
                copyObject(obj, objSchema, target_realm)
            });
        });

        // Do a second pass to add links
        source_realm_schema.forEach((objSchema) => {
            console.log("updating links in:", objSchema['name']);
            const allSourceObjects = source_realm.objects(objSchema['name']);
            const allTargetObjects = target_realm.objects(objSchema['name']);

            for (var i = 0; i < allSourceObjects.length; ++i) {
                const sourceObject = allSourceObjects[i];
                const targetObject = allTargetObjects[i];

                addLinksToObject(sourceObject, targetObject, objSchema, source_realm, target_realm);
            }
        });
    });
}


// Login to server
Realm.Sync.User.login("http://" + realm_server, username, password, (error, user) => {
    if (error) {
        console.log("Login failed", error);
        return;
    }

    const remote_realm_url = "realm://" + realm_server + target_realm_path;

    copyRealm(user, source_realm_path, remote_realm_url);

    console.log("done");
});
```
{% endtab %}
{% endtabs %}





Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

