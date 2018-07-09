# Errors

## Error Reporting

{% tabs %}
{% tab title="Swift" %}
Certain sync-related APIs perform asynchronous operations that may fail. These APIs take completion blocks which accept an error parameter; if the error parameter is passed in the operation failed. The error can be checked for more detail.

It is **strongly recommended** to also set an error handler on the `SyncManager` singleton. Errors involving the global synchronization subsystem or specific sessions \(which represent Realms opened for synchronization with the server\) will be reported through this error handler. When errors occur, the error handler will be called with an object representing the error, as well as a `SyncSession` object representing the session the error occurred on \(if applicable\).

```swift
SyncManager.shared.errorHandler = { error, session in
    // handle error
}
```

Realm Platform errors are represented by `SyncError` values, which conform to Swift’s `Error`protocol.
{% endtab %}

{% tab title="Objective-C" %}
Certain sync-related APIs perform asynchronous operations that may fail. These APIs take completion blocks which accept an error parameter; if the error parameter is passed in the operation failed. The error can be checked for more detail.

It is **strongly recommended** to also set an error handler on the `RLMSyncManager` singleton. Errors involving the global synchronization subsystem or specific sessions \(which represent Realms opened for synchronization with the server\) will be reported through this error handler. When errors occur, the error handler will be called with an object representing the error, as well as a `RLMSyncSession` object representing the session the error occurred on \(if applicable\).

```objectivec
[[RLMSyncManager sharedManager] setErrorHandler:^(NSError *error, RLMSyncSession *session) {
    // handle error
}];
```

Realm Mobile Platform errors are represented by `NSError` objects whose domain is `RLMSyncErrorDomain`. Please consult the definitions of `RLMSyncError` and `RLMSyncAuthError` for error codes and what they mean.
{% endtab %}

{% tab title="Java" %}
It is possible to set up error handling by registering an error handler:

```java
SyncConfiguration configuration = new SyncConfigurtion.Builder(user, serverURL)
  .errorHandler(new Session.ErrorHandler() {
    void onError(Session session, ObjectServerError error) {
        // do some error handling
    }
  })
  .build();
```

It is also possible to register a default global error handler that will apply to all SyncConfigurations:

```java
SyncManager.setDefaultSessionErrorHandler(myErrorHandler);
```
{% endtab %}

{% tab title="Javascript" %}
The error handling is set up by registering a callback \(`error`\) as part of the configuration:

```javascript
const config = {
  sync: { 
    user: userA,
    url: realmUrl,
    error: (error) => {
      console.log(error.name, error.message);
    },
  },
  schema: [{ name: 'Dog', properties: { name: 'string' } }]
};

var realm = new Realm(config);
```
{% endtab %}

{% tab title=".Net" %}
Realm Platform errors are represented by [SessionExceptions](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.Exceptions.SessionException.html). In addition to the standard exception properties, you have access to an [ErrorCode](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.Exceptions.ErrorCode.html) that contains information about the type of the error and allows you to have strongly typed handling logic, e.g.:

```csharp
Session.Error += (session, errorArgs) =>
{
    var sessionException = (SessionException)errorArgs.Exception;
    switch (sessionException.ErrorCode)
    {
        case ErrorCode.AccessTokenExpired:
        case ErrorCode.BadUserAuthentication:
            // Ask user for credentials
            break;
        case ErrorCode.PermissionDenied:
            // Tell the user they don't have permissions to work with that Realm
            break;
        case ErrorCode.Unknown:
            // Likely the app version is too old, prompt for update
            break;
        // ...
    }
};
```
{% endtab %}
{% endtabs %}

## Setting the client logging level 

While developing your application, you will likely need to troubleshoot unexpected behavior.  When doing this, it is useful to increase the Realm logging level within your application.  

{% tabs %}
{% tab title="Swift" %}
The synchronization subsystem supports a number of logging levels, useful while developing an app. These can be selected by setting the `logLevel` property on the `SyncManager` singleton to the desired verbosity:

```text
SyncManager.shared.logLevel = .debug
```

The logging level **must be set before any synced Realms are opened**. Changing it after the first synced Realm is opened will have no effect.
{% endtab %}

{% tab title="Objective-C" %}
The synchronization subsystem supports a number of logging levels, useful while developing an app. These can be selected by setting the `logLevel` property on the `RLMSyncManager` singleton to the desired verbosity:

```text
[[RLMSyncManager sharedManager] setLogLevel:RLMSyncLogLevelDebug];
```

The logging level **must be set before any synced Realms are opened**. Changing it after the first synced Realm is opened will have no effect.
{% endtab %}

{% tab title="Java" %}
By enabling more verbose logs, you can better see what is happening through Android `logcat`.

```text
RealmLog.setLevel(Log.DEBUG);
```
{% endtab %}

{% tab title="Javascript" %}
These can be selected by setting the `logLevel` property on `Realm.sync` to the desired verbosity. For example:

```text
Realm.Sync.setLogLevel("debug");
```
{% endtab %}

{% tab title=".Net" %}
_Coming soon_
{% endtab %}
{% endtabs %}

Available levels are as follows: 

* `fatal`
* `error`
* `warn`
* `info`
* `detail`
* `debug`
* `trace`

## Errors

### Client Reset

{% tabs %}
{% tab title="Swift" %}
If a Realm Object Server crashes and must be restored from a backup, there is a possibility that an app might need to carry out a _client reset_ on a given synced Realm. This will happen if the local version of the Realm in question is greater than the version of that same Realm on the server \(for example, if the application made changes after the Realm Object Server was backed up, but those changes weren’t synced back to the server before it crashed and had to be restored\).

The client reset procedure is as follows: a backup copy of the local Realm file is made, and then the Realm files are deleted. The next time the app connects to the Realm Object Server and opens that Realm, a fresh copy will be downloaded. Changes that were made after the Realm Object Server was backed up but weren’t synced back to the server will be preserved in the backup copy of the Realm, but will not be present when the Realm is re-downloaded.

The need for a client reset is indicated by an error sent to the `SyncManager` error handler. This error will be denoted by the code `.clientResetError`.

The error object will also contain two values: the location of the backup copy of the Realm file once the client reset process is carried out, and a token object which can be passed into the `SyncSession.immediatelyHandleError(_:)` method to manually initiate the client reset process.

If the client reset process is manually initiated, all instances of the Realm in question **must first be invalidated and destroyed**. Note that a `Realm` might not be fully invalidated, even if all references to it are nil’ed out, until the autorelease pool containing it is drained. However, doing so allows the Realm to be immediately re-opened after the client reset process is complete, allowing syncing to resume.

If the client reset process is not manually initiated, it will instead automatically take place after the next time the app is launched, upon first accessing the `SyncManager` singleton. It is the app’s responsibility to persist the location of the backup copy if needed, so that the backup copy can be found later.

A Realm which needs to be reset can still be written to and read from, but **no subsequent changes will be synced to the server until the client reset is complete and the Realm is re-downloaded**. It is extremely important that your application listen for client reset errors and, at the very least, make provisions to save user data created or modified after a client reset is triggered so it can later be written back to the re-downloaded copy of the Realm.

The backup copy file path and reset initiation token can be obtained by calling `SyncError.clientResetInfo()` on the error object.

The following example shows how the client reset APIs might be used to carry out a client reset:

```swift
SyncManager.shared.errorHandler = { error, session in
    let syncError = error as! SyncError
    switch syncError.code {
    case .clientResetError:
        if let (path, clientResetToken) = syncError.clientResetInfo() {
            closeRealmSafely()
            saveBackupRealmPath(path)
            SyncSession.immediatelyHandleError(clientResetToken)
        }
    default:
        // Handle other errors...
        ()
    }
}
```
{% endtab %}

{% tab title="Objective-C" %}
If a Realm Object Server crashes and must be restored from a backup, there is a possibility that an app might need to carry out a _client reset_ on a given synced Realm. This will happen if the local version of the Realm in question is greater than the version of that same Realm on the server \(for example, if the application made changes after the Realm Object Server was backed up, but those changes weren’t synced back to the server before it crashed and had to be restored\).

The client reset procedure is as follows: a backup copy of the local Realm file is made, and then the Realm files are deleted. The next time the app connects to the Realm Object Server and opens that Realm, a fresh copy will be downloaded. Changes that were made after the Realm Object Server was backed up but weren’t synced back to the server will be preserved in the backup copy of the Realm, but will not be present when the Realm is re-downloaded.

The need for a client reset is indicated by an error sent to the `RLMSyncManager` error handler. This error will be denoted by the code `RLMSyncErrorClientResetError`.

The error object will also contain two values: the location of the backup copy of the Realm file once the client reset process is carried out, and a token object which can be passed into the `+[RLMSyncSession immediatelyHandleError:]` method to manually initiate the client reset process.

If the client reset process is manually initiated, all instances of the Realm in question **must first be invalidated and destroyed**. Note that a `RLMRealm` might not be fully invalidated, even if all references to it are nil’ed out, until the autorelease pool containing it is drained. However, doing so allows the Realm to be immediately re-opened after the client reset process is complete, allowing syncing to resume.

If the client reset process is not manually initiated, it will instead automatically take place after the next time the app is launched, upon first accessing the `RLMSyncManager` singleton. It is the app’s responsibility to persist the location of the backup copy if needed, so that the backup copy can be found later.

A Realm which needs to be reset can still be written to and read from, but **no subsequent changes will be synced to the server until the client reset is complete and the Realm is re-downloaded**. It is extremely important that your application listen for client reset errors and, at the very least, make provisions to save user data created or modified after a client reset is triggered so it can later be written back to the re-downloaded copy of the Realm.

The backup copy file path can be obtained by calling `-[NSError rlmSync_clientResetBackedUpRealmPath]`upon the `NSError` object. It can also be extracted directly from the `userInfo` dictionary using the `kRLMSyncPathOfRealmBackupCopyKey` key.

The reset initiation token can be obtained by calling `-[NSError rlmSync_errorActionToken]` upon the `NSError` object. It can also be extracted directly from the `userInfo` dictionary using the `kRLMSyncErrorActionTokenKey` key.

The following example shows how the client reset APIs might be used to carry out a client reset:

```objectivec
[[RLMSyncManager sharedManager] setErrorHandler:^(NSError *error, RLMSyncSession *session) {
    if (error.code == RLMSyncErrorClientResetError) {
        [realmManager closeRealmSafely];
        [realmManager saveBackupRealmPath:[error rlmSync_clientResetBackedUpRealmPath]];
        [RLMSyncSession immediatelyHandleError:[error rlmSync_errorActionToken]];
        return;
    }
    // Handle other errors...
}];
```
{% endtab %}

{% tab title="Java" %}
{% hint style="warning" %}
_Details coming soon!_
{% endhint %}
{% endtab %}

{% tab title="Javascript" %}
If a Realm Object Server crashes and must be restored from a backup, there is a possibility that an app might need to carry out a client reset on a given synced Realm. This will happen if the local version of the Realm in question is greater than the version of that same Realm on the server \(for example, if the application made changes after the Realm Object Server was backed up, but those changes weren’t synced back to the server before it crashed and had to be restored\).

The client reset procedure is as follows: a backup copy of the local Realm file is made, and then the Realm files are deleted. The next time the app connects to the Realm Object Server and opens that Realm, a fresh copy will be downloaded. Changes that were made after the Realm Object Server was backed up but weren’t synced back to the server will be preserved in the backup copy of the Realm, but will not be present when the Realm is re-downloaded.

The need for a client reset is indicated by an error sent to the registered error handler. The property `name` will be `ClientReset`.

The error object will also contain two values: the location of the backup copy of the Realm file once the client reset process is carried out, and a token object which can be passed into the Realm.Sync.initiateClientReset method to manually initiate the client reset process.

If the client reset process is manually initiated, all instances of the Realm in question **must first be invalidated and destroyed**. Note that a `Realm` might not be fully invalidated, even if all references to it are nil’ed out, until the autorelease pool containing it is drained. However, doing so allows the Realm to be immediately re-opened after the client reset process is complete, allowing syncing to resume.

If the client reset process is not manually initiated, it will instead automatically take place after the next time the app is launched. It is the app’s responsibility to persist the location of the backup copy if needed, so that the backup copy can be found later.

A Realm which needs to be reset can still be written to and read from, but **no subsequent changes will be synced to the server until the client reset is complete and the Realm is re-downloaded**. It is extremely important that your application listen for client reset errors and, at the very least, make provisions to save user data created or modified after a client reset is triggered so it can later be written back to the re-downloaded copy of the Realm.

The following example shows how the client reset APIs might be used to carry out a client reset:

```javascript
config.sync.error = (error) => {
  if (error.name === 'ClientReset') {
    const path = realm.path;
    realm.close();
    Realm.Sync.initializeClientReset(path);
    // copy required objects from Realm at error.config.path
  }
};
```
{% endtab %}

{% tab title=".Net" %}
If a Realm Object Server crashes and must be restored from a backup, there is a possibility that an app might need to carry out a _client reset_ on a given synced Realm. This will happen if the local version of the Realm in question is greater than the version of that same Realm on the server \(for example, if the application made changes after the Realm Object Server was backed up, but those changes weren’t synced back to the server before it crashed and had to be restored\).

The client reset procedure is as follows: a backup copy of the local Realm file is made, and then the Realm files are deleted. The next time the app connects to the Realm Object Server and opens that Realm, a fresh copy will be downloaded. Changes that were made after the Realm Object Server was backed up but weren’t synced back to the server will be preserved in the backup copy of the Realm, but will not be present when the Realm is re-downloaded.

The need for a client reset is indicated by an error sent to [Session.Error](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.Session.html#Realms_Sync_Session_Error) subscribers. You can check the error type by examining the result of [sessionException.ErrorCode.IsClientResetError](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.Exceptions.ErrorCodeExtensions.html#Realms_Sync_Exceptions_ErrorCodeExtensions_IsClientResetError_Realms_Sync_Exceptions_ErrorCode_) or safe-casting the exception to [ClientResetException](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.Exceptions.ClientResetException.html). This type contains an additional property - [BackupFilePath](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.Exceptions.ClientResetException.html#Realms_Sync_Exceptions_ClientResetException_BackupFilePath) - that contains the location of the backup copy of the Realm file once the client reset process is carried out, as well as a method - [InitiateClientReset](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.Exceptions.ClientResetException.html#Realms_Sync_Exceptions_ClientResetException_InitiateClientReset) which can be called to initiate the client reset process.

If the method is called to manually initiate the client reset process, all instances of the Realm in question **must first be closed by calling Dispose\(\)** before the method is invoked. However, doing so allows the Realm to be immediately re-opened after the client reset process is complete, allowing syncing to resume.

If [InitiateClientReset](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.Exceptions.ClientResetException.html#Realms_Sync_Exceptions_ClientResetException_InitiateClientReset) is not called, the client reset process will automatically take place after the next time the app is launched, upon first obtaining a [Realm](https://realm.io/docs/dotnet/latest/api/reference/Realms.Realm.html) instance. It is the app’s responsibility to persist the location of the backup copy if needed, so that the backup copy can be found later.

A Realm which needs to be reset can still be written to and read from, but **no subsequent changes will be synced to the server until the client reset is complete and the Realm is re-downloaded**. It is extremely important that your application listen for client reset errors and, at the very least, make provisions to save user data created or modified after a client reset is triggered so it can later be written back to the re-downloaded copy of the Realm.

The following example shows how the client reset APIs might be used to carry out a client reset:

```csharp
void CloseRealmSafely()
{
    // Safely dispose the realm instances, possibly notify the user
}

void SaveBackupRealmPath(string path)
{
    // Persist the location of the backup realm
}

void SetupErrorHandling()
{
    Session.Error += (session, errorArgs)
    {
        var sessionException = (SessionException)errorArgs.Exception;
        if (sessionException.ErrorCode.IsClientResetError())
        {
            var clientResetException = (ClientResetException)errorArgs.Exception;
            CloseRealmSafely();
            SaveBackupRealmPath(clientResetException.BackupFilePath);
            clientResetException.InitiateClientReset();
        }

        // Handle other errors
    }
}
```
{% endtab %}
{% endtabs %}

### Permission Denied

{% tabs %}
{% tab title="Swift" %}
If a user attempts to perform operations on a Realm for which they do not have appropriate permissions, a permission denied error will be reported. Such an error may occur, for example, if a user attempts to write to a Realm for which they only have read permissions or opens a Realm for which they have no permissions at all.

It is important to note that if a user has only read access to a particular synchronized Realm they must open that Realm asynchronously using the `Realm.asyncOpen(configuration:, callbackQueue:, callback:)`API. Failure to do so will lead to a permission denied error being reported.

A permission denied error will be denoted by the code `.permissionDeniedError`. The error object will contain an opaque token which can be used to delete the offending Realm file. The token can be obtained by calling `SyncError.deleteRealmUserInfo()` on the error object.

In all cases, a permission denied error indicates that the local copy of the Realm on the user’s device has diverged in a way that cannot be reconciled with the server. The file will be automatically deleted the next time the application is launched. The token included with the error can also be passed into the `SyncSession.immediatelyHandleError(_:)` method in order to immediately delete the Realm files.

If the token is used to immediately delete the Realm files, all instances of the Realm in question **must first be invalidated and destroyed**. Note that a `Realm` might not be fully invalidated, even if all references to it are nil’ed out, until the autorelease pool containing it is drained. However, doing so allows the Realm to be immediately re-opened in the correct way or after the correct permissions have been set, allowing syncing to resume.
{% endtab %}

{% tab title="Objective-C" %}
If a user attempts to perform operations on a Realm for which they do not have appropriate permissions, a permission denied error will be reported. Such an error may occur, for example, if a user attempts to write to a Realm for which they only have read permissions or opens a Realm for which they have no permissions at all.

It is important to note that if a user has only read access to a particular synchronized Realm they must open that Realm asynchronously using the `+[RLMRealm asyncOpenWithConfiguration:callbackQueue:callback:]` API. Failure to do so will lead to a permission denied error being reported.

A permission denied error will be denoted by the code `RLMSyncErrorPermissionDeniedError` . The error object will contain an opaque token which can be used to delete the offending Realm file. The token can be obtained by calling `-[NSError rlmSync_errorActionToken]` upon the `NSError` object. It can also be extracted directly from the `userInfo` dictionary using the `kRLMSyncErrorActionTokenKey` key.

In all cases, a permission denied error indicates that the local copy of the Realm on the user’s device has diverged in a way that cannot be reconciled with the server. The file will be automatically deleted the next time the application is launched. The token included with the error can also be passed into the `+[RLMSyncSession immediatelyHandleError:]` method in order to immediately delete the Realm files.

If the token is used to immediately delete the Realm files, all instances of the Realm in question **must first be invalidated and destroyed**. Note that a `RLMRealm` might not be fully invalidated, even if all references to it are nil’ed out, until the autorelease pool containing it is drained. However, doing so allows the Realm to be immediately re-opened in the correct way or after the correct permissions have been set, allowing syncing to resume.
{% endtab %}

{% tab title="Java" %}
{% hint style="warning" %}
_Details coming soon!_
{% endhint %}
{% endtab %}

{% tab title="Javascript" %}
{% hint style="warning" %}
_Details coming soon!_
{% endhint %}
{% endtab %}

{% tab title=". Net" %}
If a user attempts to perform operations on a Realm for which they do not have appropriate permissions, a permission denied error will be reported. Such an error may occur, for example, if a user attempts to write to a Realm for which they only have read permissions or opens a Realm for which they have no permissions at all.

It is important to note that if a user has only read access to a particular synchronized Realm they must open that Realm asynchronously using the `Realm.GetInstanceAsync` API. Failure to do so will lead to a permission denied error being reported.

A permission denied error will be denoted by the code `ErrorCode.PermissionDenied` and its runtime type will be `PermissionDeniedException`. In all cases, this error indicates that the local copy of the Realm can no longer synchronize with the remote one and will be automatically deleted the next time the application is launched. If you want to perform the deletion immediately so you can reopen the Realm, you can invoke the `PermissionDeniedException.DeleteRealmUserInfo` method with an argument `deleteRealm: true`. Be advised that all references to that Realm must be disposed of prior to invoking the method. If you want to cancel the deletion, you can pass `deleteRealm: false`.
{% endtab %}
{% endtabs %}

### Session Specific Errors

| **Error Message** | **Cause** |
| :--- | :--- |
| 204 “Illegal Realm path \(BIND\)” | Indicates that the Realm path is not valid for the user. |
| 207 or 208 “Bad server file identifier \(IDENT\)” | Indicates that the local Realm specifies a link to a server-side Realm that does not exist. This is most likely because the server state has been completely reset. |
| 211 “Diverging histories \(IDENT\)” | Indicates that the local Realm specifies a server version that does not exists. This is most likely because the server state has been partially reset \(for example because a backup was restored\). |

### Client Level Errors

| **Error Message** | **Cause** |
| :--- | :--- |
| 105 “Wrong protocol version \(CLIENT\)” | The client and the server use different versions of the sync protocol due to a mismatch in upgrading. For example: "client protocol version = 24, server protocol version = 22". Refer to [Version Compatibilities](version-compatibilities.md) table to ensure client and server are compatible. |
| 108 “Client file bound in other session \(IDENT\)” | Indicates that multiple sync sessions for the same client-side Realm file overlap in time. |
| 203 “Bad user authentication \(BIND, REFRESH\)” | Indicates that the server has produced a bad token, or that the SDK has done something wrong. |
| 206 “Permission denied \(BIND, REFRESH\)” | Indicates that the user does not have permission to access the Realm at the given path. |

### Operational Errors

Occasionally it can be useful for debugging purposes to check the Realm Object Server logs - which are output to the terminal by default but can easily be output to the file system or an external logger by extending your `index.ts`. Realm Object Server produces two specific classes of error and warning diagnostics that may be useful to system admins and developers.

| **Error Message** | **Solution** |
| :--- | :--- |
| Failed to accept a connection due to the file descriptor limit, consider increasing the limit in your system config | Increase the file descriptor limit on your machine.  Instructions on how to do this can be found [here](https://www.tecmint.com/increase-set-open-file-limits-in-linux/).   |
| mmap\(\) failed: Cannot allocate memory  | Consider increasing the amount of memory available on your server.  If you believe that your server has ample resources, please [contact us](https://support.realm.io/).   |

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

