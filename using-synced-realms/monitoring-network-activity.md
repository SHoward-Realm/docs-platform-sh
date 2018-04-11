# Monitoring Network Activity

## Overview

When you [open a synchronized Realm](opening-a-synced-realm.md), it will establish a network connection with the server in the background. Realm Platform uses Websockets with a custom binary protocol designed to be highly efficient for real-time performance. The Cloud version uses SSL/TLS by default, whereas with the Self-Hosted version this requires setup.

## Sync Session

The client APIs provide access to the underlying networking through a session object. This can be used to query for the networking state.

{% tabs %}
{% tab title="Swift" %}
Session objects representing Realms opened by a specific user can be retrieved from that user’s `SyncUser` object using the `SyncUser.allSessions()` or `SyncUser.session(for:)` APIs.

The state of the underlying session can be retrieved using the `state` property. This can be used to check whether the session is active, not connected to the server, or in an error state. If the session is valid, the `configuration` property will contain a `SyncConfiguration` value that can be used to open another instance of the same Realm \(for example, on a different thread\).
{% endtab %}

{% tab title="Objective-C" %}
A synced Realm’s connection to the Realm Object Server is represented by a `RLMSyncSession` object. Session objects representing Realms opened by a specific user can be retrieved from that user’s `RLMSyncUser` object using the `+[RLMSyncUser allSessions]` or `-[RLMSyncUser sessionForURL:]` APIs.

The state of the underlying session can be retrieved using the `state` property. This can be used to check whether the session is active, not connected to the server, or in an error state. If the session is valid, the `configuration` property will contain a `RLMSyncConfiguration` value that can be used to open another instance of the same Realm \(for example, on a different thread\).
{% endtab %}

{% tab title="Java" %}
A synced Realm’s connection to the Realm Object Server is represented by a `SyncSession` object. A session object for a specific Realm can be retrieved by using the `SyncManager.getSession(SyncConfiguration)` API.
{% endtab %}

{% tab title="Javascript" %}
A synced Realm’s connection to the Realm Object Server is represented by a `Session` object. Session objects can be retrieved by calling `realm.syncSession`.

The state of the underlying session can be retrieved using the `state` property. This can be used to check whether the session is active, not connected to the server, or in an error state.
{% endtab %}

{% tab title=".Net" %}
A synced Realm’s connection to the Realm Object Server is represented by a [Session](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.Session.html) object. Session objects can be retrieved by calling [realm.GetSession\(\)](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.RealmSyncExtensions.html#Realms_Sync_RealmSyncExtensions_GetSession_Realms_Realm_).

The state of the underlying session can be retrieved using the [State](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.Session.html#Realms_Sync_Session_State) property. This can be used to check whether the session is active, not connected to the server, or in an error state.
{% endtab %}
{% endtabs %}

## Progress Notifications

{% tabs %}
{% tab title="Swift" %}
Session objects allow your app to monitor the status of a session’s uploads to and downloads from the Realm Object Server by registering _progress notification blocks_ on a session object.

Progress notification blocks will be invoked periodically by the synchronization subsystem on the runloop of the thread in which they were originally registered. If no runloop exists, one will be created. \(Practically speaking, this means you can register these blocks on background queues in GCD and they will work fine.\) As many blocks as needed can be registered on a session object simultaneously. Blocks can either be configured to report upload progress or download progress.

Each time a block is called, it will receive the current number of bytes already transferred, as well as the total number of transferrable bytes \(defined as the number of bytes already transferred plus the number of bytes pending transfer\).

When a block is registered, the registration method returns a token object. The `invalidate()` method can be called on the token to stop notifications for that particular block. If the block has already been deregistered, calling the stop method does nothing. Note that the registration method might return a nil token if the notification block will never run again \(for example, because the session was already in a fatal error state, or there is no further progress to report\).

There are two types of blocks. Blocks can be configured to _report indefinitely_. These blocks will remain active unless explicitly stopped by the user and will always report the most up-to-date number of transferrable bytes. This type of block could be used to control a network indicator UI that, for example, changes color or appears only when uploads or downloads are actively taking place.

```swift
let session = SyncUser.current!.session(for: realmURL)!
let token = session.addProgressNotification(for: .download,
                                            mode: .reportIndefinitely) { progress in
    if progress.isTransferComplete {
        hideActivityIndicator()
    } else {
        showActivityIndicator()
    }
}

// Much later...
token?.invalidate()
```

Blocks can also be configured to _report progress for currently outstanding work_. These blocks capture the number of transferrable bytes at the moment they are registered and always report progress relative to that value. Once the number of transferred bytes reaches or exceeds that initial value, the block will automatically unregister itself. This type of block could, for example, be used to control a progress bar that tracks the progress of an initial download of a synced Realm when a user signs in, letting them know how long it is before their local copy is up-to-date.

```swift
let session = SyncUser.current!.session(for: realmURL)!
var token: SyncSession.ProgressNotificationToken?
token = session.addProgressNotification(for: .upload,
                                        mode: .forCurrentlyOutstandingWork) { progress in
    updateProgressBar(fraction: progress.fractionTransferred)
    if progress.isTransferComplete {
        hideProgressBar()
        token?.invalidate()
    }
}
```
{% endtab %}

{% tab title="Objective-C" %}
Session objects allow your app to monitor the status of a session’s uploads to and downloads from the Realm Object Server by registering _progress notification blocks_ on a session object.

Progress notification blocks will be invoked periodically by the synchronization subsystem on the runloop of the thread in which they were originally registered. If no runloop exists, one will be created. \(Practically speaking, this means you can register these blocks on background queues in GCD and they will work fine.\) As many blocks as needed can be registered on a session object simultaneously. Blocks can either be configured to report upload progress or download progress.

Each time a block is called, it will receive the current number of bytes already transferred, as well as the total number of transferrable bytes \(defined as the number of bytes already transferred plus the number of bytes pending transfer\).

When a block is registered, the registration method returns a token object. The `-invalidate` method can be called on the token to stop notifications for that particular block. If the block has already been deregistered, calling the stop method does nothing. Note that the registration method might return a nil token if the notification block will never run again \(for example, because the session was already in a fatal error state, or there is no further progress to report\).

There are two types of blocks. Blocks can be configured to _report indefinitely_. These blocks will remain active unless explicitly stopped by the user and will always report the most up-to-date number of transferrable bytes. This type of block could be used to control a network indicator UI that, for example, changes color or appears only when uploads or downloads are actively taking place.

```objectivec
RLMSyncSession *session = [[RLMSyncUser currentUser] sessionForURL:realmURL];
void(^workBlock)(NSUInteger, NSUInteger) = ^(NSUInteger downloaded, NSUInteger downloadable) {
    if ((double)downloaded/(double)downloadable >= 1) {
        [viewController hideActivityIndicator];
    } else {
        [viewController showActivityIndicator];
    }
};
RLMProgressNotificationToken *token;
token = [session addProgressNotificationForDirection:RLMSyncProgressDirectionDownload
                                                mode:RLMSyncProgressModeReportIndefinitely
                                               block:workBlock];

// Much later...
[token invalidate];
```

Blocks can also be configured to _report progress for currently outstanding work_. These blocks capture the number of transferrable bytes at the moment they are registered and always report progress relative to that value. Once the number of transferred bytes reaches or exceeds that initial value, the block will automatically unregister itself. This type of block could, for example, be used to control a progress bar that tracks the progress of an initial download of a synced Realm when a user signs in, letting them know how long it is before their local copy is up-to-date.

```objectivec
RLMSyncSession *session = [[RLMSyncUser currentUser] sessionForURL:realmURL];
RLMProgressNotificationToken *token;
void(^workBlock)(NSUInteger, NSUInteger) = ^(NSUInteger uploaded, NSUInteger uploadable) {
    double progress = ((double)uploaded/(double)uploadable);
    progress = progress > 1 ? 1 : progress;
    [viewController updateProgressBarWithFraction:progress];
    if (progress == 1) {
        [viewController hideProgressBar];
        [token invalidate];
    }
};
token = [session addProgressNotificationForDirection:RLMSyncProgressDirectionUpload
                                                mode:RLMSyncProgressModeForCurrentlyOutstandingWork
                                               block:workBlock];
```
{% endtab %}

{% tab title="Java" %}
Session objects allow your app to monitor the status of a session’s uploads to and downloads from the Realm Object Server by registering _progress notifications_ on a session object.

Progress notifications are invoked periodically by the synchronization subsystem. The notification callback will happen on a worker thread, so manipulating UI elements must be done using `Activity.runOnUiThread`or similar. As many notifications as needed can be registered on a session object simultaneously. Notifications can either be configured to report upload progress or download progress.

Each time a notification is called, it will receive the number of bytes already transferred, as well as the total number of transferrable bytes \(the number of bytes already transferred plus the number of bytes pending transfer\).

Notifications can be un-registered by using `SyncSession.removeProgressListener`.

There are two types of notification modes: `ProgressMode.CURRENT_CHANGES` and `ProgressMode.INDEFINITELY`. Notifications configured with `ProgressMode.INDEFINITELY` will remain active unless explicitly stopped by the user and will always report the most up-to-date number of transferrable bytes. This type of block could be used to control a network indicator UI that, for example, changes color or appears only when uploads or downloads are actively taking place. A notification in this mode can report `Progress.isTransferComplete` multiple times.

```java
ProgressListener listener = new ProgressListener() {
    @Override
    public void onChange(Progress progress) {
        activity.runOnUiThread(new Runnable) {
            @Override
            public void run() {
                if (progress.isTransferComplete()) {
                    hideActivityIndicator();
                } else {
                    showActivityIndicator();
                }
            }
        }
    }
};

SyncSession session = SyncManager.getSession(getSyncConfiguration());
session.addDownloadProgressListener(ProgressMode.INDEFINITELY, listener);

// When stopping activity
session.removeProgressListener(listener);
```

Notifications configured with `ProgressMode.CURRENT_CHANGES` only report progress for _currently outstanding work_. These notifications capture the number of transferrable bytes at the moment they are registered and always report progress relative to that value. Once the number of transferred bytes reaches or exceeds that initial value, the notification will send one final event where `Progress.isTransferComplete` is `true` and then never again. The listener should be unregistered at this point. This type of notification could, for example, be used to control a progress bar that tracks the progress of an initial download of a synced Realm when a user signs in, letting them know when their local copy is up-to-date.

```java
final SyncSession session = SyncManager.getSession(getSyncConfiguration());
ProgressListener listener = new ProgressListener() {
    @Override
    public void onChange(Progress progress) {
        activity.runOnUiThread(new Runnable) {
            @Override
            public void run() {
                if (progress.isTransferComplete()) {
                    hideProgressBar();
                    session.removeProgressListener(this);
                } else {
                    updateProgressBar(progress.getFractionTransferred());
                }
            }
        }
    }
};
setupProgressBar();
session.addDownloadProgressListener(ProgressMode.CHANGES_ONLY, listener);
```
{% endtab %}

{% tab title="Javascript" %}
Session objects allow your app to monitor the status of a session’s uploads to and downloads from the Realm Object Server by registering a progress notification callback on a session object by calling \[realm.syncSession.addProgressNotification\(direction, mode, callback\)\] or one of the asynchronous methods `Realm.open` and `Realm.openAsync`\(_these methods support only a subset of the progress notifications modes_\).

Progress notification callbacks will be invoked periodically by the synchronization subsystem. As many callbacks as needed can be registered on a session object simultaneously. Callbacks can either be configured to report upload progress or download progress.

Each time a callback is called, it will receive the current number of bytes already transferred, as well as the total number of transferable bytes \(defined as the number of bytes already transferred plus the number of bytes pending transfer\).

To stop receiving progress notifications a callback can be unregistered using \[realm.syncSession.removeProgressNotification\(callback\)\]. Calling the function a second time with the same callback is ignored.

There are two [progress notification modes](https://realm.io/docs/javascript/latest/api/reference/Realms.Sync.ProgressMode.html#fields) for the progress notifications:

* `reportIndefinitely` - the registration will stay active until the callback is unregistered and will always report the most up-to-date number of transferable bytes. This type of callback could be used to control a network indicator UI that, for example, changes color or appears only when uploads or downloads are actively taking place.

```javascript
let realm = new Realm(config);
const progressCallback = (transferred, transferables) => {
    if (transferred < transferables) {
        // Show progress indicator
    } else {
        // Hide the progress indicator
    }
};

realm.syncSession.addProgressNotification('upload', 'reportIndefinitely', progressCallback);
// ...
realm.syncSession.removeProgressNotification(progressCallback);
```

* `forCurrentlyOutstandingWork` - the registration will capture the number of transferable bytes at the moment it is registered and always report progress relative to that value. Once the number of transferred bytes reaches or exceeds that initial value, the callback will be automatically unregistered. This type of progress notification could, for example, be used to control a progress bar that tracks the progress of an initial download of a synced Realm when a user signs in, letting them know how long it is before their local copy is up-to-date.

```javascript
let realm = new Realm(config);
const progressCallback = (transferred, transferable) => {
    const progressPercentage = transferred / transferable;
};

realm.syncSession.addProgressNotification('download', 'forCurrentlyOutstandingWork', progressCallback);
// ...
realm.syncSession.removeProgressNotification(progressCallback);
```

The asynchronous methods `Realm.open` and `Realm.openAsync` for opening a Realm can also be used to register a callback for sync progress notifications. In this case only ‘download’ direction and ‘forCurrentlyOutstandingWork’ mode are supported. Additional callbacks can be registered after the initial open is completed by using the newly created Realm instance.

```javascript
Realm.open(config)
      .progress((transferred, transferable) => {
          // update UI
      })
      .then(realm => {
          // use the Realm
      })
      .catch((e) => reject(e));

Realm.openAsync(config,
          (error, realm) => {
             // use the Realm or report an error
          },
          (transferred, transferable) => {
            // update UI
          }, );
```
{% endtab %}

{% tab title=".Net" %}
Session objects allow your app to monitor the status of a session’s uploads to and downloads from the Realm Object Server by registering a subscriber on the [IObservable](https://msdn.microsoft.com/en-us/library/dd990377%28v=vs.110%29.aspx) instance, obtained by calling [session.GetProgressObservable\(direction, mode\)](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.Session.html#Realms_Sync_Session_GetProgressObservable_Realms_Sync_ProgressDirection_Realms_Sync_ProgressMode_).

The subscription callback will be invoked periodically by the synchronization subsystem. As many subscribers as needed can be registered on a session object simultaneously. Subscribers can either be configured to report upload progress or download progress. Each time a subscriber is called, it will be passed a [SyncProgress](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.SyncProgress.html) instance that will contain information about the current number of bytes already transferred, as well as the total number of transferable bytes \(defined as the number of bytes already transferred plus the number of bytes pending transfer\).

Each time a subscriber is registered, an [IDisposable](https://msdn.microsoft.com/en-us/library/system.idisposable%28v=vs.110%29.aspx) token will be returned. You must keep a reference to that token for as long as progress notifications are desired. To stop receiving notifications, call [Dispose](https://msdn.microsoft.com/en-us/library/es4s3w1d%28v=vs.110%29.aspx) on the token.

There are two [modes](https://realm.io/docs/dotnet/latest/api/reference/Realms.Sync.ProgressMode.html#fields) for the progress subscriptions: 

* `ReportIndefinitely` - the subscription will stay active until `Dispose` is explicitly called and will always report the most up-to-date number of transferrable bytes. This type of callback could be used to control a network indicator UI that, for example, changes color or appears only when uploads or downloads are actively taking place.

```csharp
var token = session.GetProgressObservable(ProgressDirection.Download, ProgressMode.ReportIndefinitely)
                   .Subscribe(progress =>
                   {
                       if (progress.TransferredBytes < progress.TransferableBytes)
                       {
                           // Show progress indicator
                       }
                       else
                       {
                           // Hide the progress indicator
                       }
                   });
```

* `ForCurrentlyOutstandingWork` - the subscription will capture the number of transferable bytes at the moment it is registered and always report progress relative to that value. Once the number of transferred bytes reaches or exceeds that initial value, the subscriber will automatically unregister itself. This type of subscription could, for example, be used to control a progress bar that tracks the progress of an initial download of a synced Realm when a user signs in, letting them know how long it is before their local copy is up-to-date.

```csharp
var token = session.GetProgressObservable(ProgressDirection.Download, ProgressMode.ForCurrentlyOutstandingWork)
                   .Subscribe(progress =>
                   {
                       var progressPercentage = progress.TransferredBytes / progress.TransferableBytes;
                       progressBar.SetValue(progressPercentage);

                       if (progressPercentage == 1)
                       {
                           progressBar.Hide();
                       }
                   });
```

Note that those examples use the [ObservableExtensions.Subscribe](https://msdn.microsoft.com/en-us/library/ff402849%28v=vs.103%29.aspx) extension method to simplify subscribing. We recommend using the [Reactive Extensions](https://github.com/Reactive-Extensions/Rx.NET) class library as it greatly simplifies working with observable sequences. For example, here’s a more advanced scenario, where both upload and download progress is tracked, throttled, and finally, dispatched on the main thread:

```csharp
var uploadProgress = session.GetProgressObservable(ProgressDirection.Upload, ProgressMode.ReportIndefinitely);
var downloadProgress = session.GetProgressObservable(ProgressDirection.Download, ProgressMode.ReportIndefinitely);

var token = uploadProgress.CombineLatest(downloadProgress, (upload, download) =>
                          {
                              return new
                              {
                                  TotalTransferred = upload.TransferredBytes + download.TransferredBytes,
                                  TotalTransferable = upload.TransferableBytes + download.TransferableBytes
                              };
                          })
                          .Throttle(TimeSpan.FromSeconds(0.1))
                          .ObserveOn(SynchronizationContext.Current)
                          .Subscribe(progress =>
                          {
                              if (progress.TotalTransferred < progress.TotalTransferable)
                              {
                                  // Show spinner
                              }
                              else
                              {
                                  // Hide spinner
                              }
                          });
```
{% endtab %}
{% endtabs %}

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

