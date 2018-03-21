# Backup

The Realm Object Server provides one or two backup systems, depending on your Edition.

* The **manual backup** system is included in all versions of the Realm Platform via a command line utility. It can be triggered during server operations to create a copy of the data in the Object Server.
* The **synchronous backup** system, available in the Professional and Enterprise Editions, is a backup slave running alongside the master Realm Object Server. The backup slave continuously receives changes across all Realms.

Both systems create a data directory from which the Realm Object Server can be restarted after a server crash. The backed up data consists of the keys, the user Realms, user account information, and all other metadata used by the Realm Object Server. Backup can be made from a running Realm Object Server without taking it offline.

**Manual backup**

The manual backup is a console command that backs up a running instance of the Realm Object Server. This command can be executed on a server without shutting it down, any number of times, in order to create an “at rest” backup that can be stored on long-term storage for safekeeping. In order to be as agnostic as possible regarding the way the backup will be persisted, the command simply creates a directory structure containing everything the server needs to get started again in the event of a disk failure.

Because Realms can be modified during the backup process, the backup command uses Realm’s transaction features to take a consistent snapshot of each Realm. However, since the server is running continuously, the backed up Realms do not represent the state of the server at one particular instant in time. A Realm added to the server while a backup is in progress might be completely left out of the backup. Such a Realm will be included in the next backup.

To run `ros backup` at the command line, type:

```bash
ros backup -f SOURCE -t TARGET
```

* `SOURCE` is the data directory of the Realm Object Server.
* `TARGET` is the directory where the backup files will be placed. This directory must be empty or absent when the backup starts for safety reasons.

After the backup command completes, `TARGET` will be a directory containing all keys and Realms needed for backup.

**Server Recovery From A Backup**

If the data of a Realm Object Server is lost or otherwise corrupted, a new Realm Object Server can be restarted with the backed up data. This is done simply by stopping the server, if needed, and placing the latest backup directory into the Realm Object Server’s data directory. After the backup has been fully placed in the Realm Object Server’s data directory, the server may be restarted.

**Client Recovery From A Backup**

Any data added to the Object Server _after_ its most recent backup will be lost when the server is restored from that backup. Since Realm Database clients communicate with the Realm Object Server continuously, though, it’s possible for clients to have been updated with that newer data that’s no longer on the server.

In the case of such an inconsistency, the Realm Object Server will send an error message to the client, and refuse to synchronize data.

The client side APIs will emit a “client reset” exception or error and give the app the possibility to reset the Realm and download the data from the server.

If client reset happens, the client will lose any data that was not included in the backup data. The best way to minimize this potential data loss is to use the synchronous backup system. If you can’t do that, run the manual backup system frequently.

Not what you were looking for? [Leave Feedback](mailto:docs-feedback@realm.io)

