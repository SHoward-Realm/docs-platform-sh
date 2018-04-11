# Upgrading

**Migrating from ROS 1.x**

There is a built-in migration tool in the CLI which converts the old ROS \(1.x\) metadata into the new schema and \(optionally\) copies the user Realms to the new location. The user Realms are not converted by this tool, because ROS itself performs the necessary low-level conversion of the Realms when it starts.

{% hint style="info" %}
_In the examples we assume that the old server is running in a root dir_ `/srv/old-root` _and is using keys_ `/srv/keys/auth.key` _and_ `/srv/keys/auth.pub` _._
{% endhint %}

Migration consists of the following steps.

1. Stop ROS 1.x
2. Run the migration tool

   ```bash
   mkdir -p /srv/new-root # should be an empty dir

   ros migrate --from /srv/old-root --to /srv/new-root/ --copyrealms
   ```

   If you use custom paths in the ROS 1.x configuration then omit `--copyrealms`and copy the user Realms manually since the CLI doesn’t support configuring the custom path.

3. Copy the keys

   Skip this if you intend to store the keys in a different place and point to them with `--private-key` and `--public-key` flags.

   ```bash
   KEYDIR=/srv/new-ros/keys # ROS 2.x uses 'keys' subdir by default
   mkdir -p /srv/new-ros/keys
   cp /srv/keys/auth.{key,pub} /srv/new-ros/keys/
   ```

4. Start ROS 2.x

   ```bash
   ros start -d /srv/new-ros
   ```

**Compatibility With 1.x**

The 2.0 server is not backwards compatible with 1.x-compatible client SDKs. This means that if you upgrade your server in production, apps running 1.x-compatible SDKs will stop syncing until they are updated to 2.0-compatible SDKs.

_**What is the cause of the backwards compatibility?**_

The unique capability of Realm Platform is our ability to sync data changes that occurred offline seamlessly. The algorithms powering this functionality are novel and we identified a way to drastically improve the performance when merging out-of-order operations. These optimizations required that we add a unique and stable identifier to every object. This identifier is internal \(we might expose in the future given that applications could utilize it\), so it doesn’t change the outward experience at all, but at a protocol-level this is where the lack of backwards compatibility comes from.

**Upgrading Deployed 1.x app**

{% hint style="info" %}
_**These instructions are subject to change but are provided for reference.**_
{% endhint %}

The general steps include:

* Deploy a new version of the app using a 1.x-compatible SDK that has logic to react to an error callback triggered when the app connects to a 2.0 server. This logic should display UI to inform the user to upgrade the app.
* Allow the user base to upgrade to this new version.
* Submit a new version of the app using a 2.0-compatible SDK to the App Store, but prevent it from immediate public release.
* Once the 2.0-compatible version of the app is available for public release, plan the upgrade of the Realm Object Server to 2.0 alongside the public release of the 2.0 compatible app in the App Store.

The resulting user experience should be that a user on the 1.x version of the app will connect to the newly upgraded 2.0 server and receive visual instructions in the app to go to the App Store to update to continue syncing. Because the 2.0 app in the App Store was made available publicly alongside the upgrade, the user should be able to immediate update and connect to the new server.

For apps deployed through enterprise App Stores that can control when the app gets updated, this process will be simpler. The complicated nature is due to the fact that developers of apps on the public App Stores have less control over their users’ upgrade process.

Finally, we can provide custom workarounds to enable running two Realm Object Servers in parallel to further ease the upgrade experience for end-users for paid customers.

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

