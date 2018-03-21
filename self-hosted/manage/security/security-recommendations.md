# Security Recommendations

The Realm Database can be configured to encrypt data at rest when using Realm on the mobile device. Enabling encryption of a realm file is as simple as adding a single line of code to the realm configuration. iOS, macOS, tvOS and watchOS versions of Realm use the CommonCrypto library whereas the Windows uses the built-in Crypto library and Android platforms use OpenSSL. Once a realm file has been created with an encryption key, that key will then be required every time that file is subsequently opened. If any attempts are made to open the file with the incorrect key \(or no key at all\), Realm will simply return an error stating that the database was ‘invalid’.

The User supplies a 64 byte encryption key. We recommend that this key is stored in the secure keychain or keystore of the device, of which most newer devices provide APIs backed by dedicated secure hardware \(see Apple’s [security white paper](https://www.apple.com/business/docs/iOS_Security_Guide.pdf) or Android’s [security white paper](https://static.googleusercontent.com/media/enterprise.google.com/en//android/static/files/android-for-work-security-white-paper.pdf) for more information\). The first 32 bytes are used for encryption. The next 24 bytes are used for the signature, with the remaining 8 bytes not used currently. In terms of how specifically we approach encryption: each 4KB block of data is encrypted with AES-256 using cipher block chaining \(CBC\) mode and a unique initialization vector \(IV\) which is never reused within a file, and then signed with a SHA-2 HMAC.

This strategy covers most attack vectors. The only option left for an attacker is to try to gain access to the pieces of decrypted data that is temporarily cached in-memory. On non-jailbroken devices there are operating system safeguards against this and in general this issue exists irrelevant of where the data originated from. In order to combat devices that can be jailbroken or rooted, giving an actor root-level privileges to the device, several Mobile Device Management \(MDM\) software providers or 3rd party libraries can be used. These give the ability to detect this level of corruption and remove sensitive data and apps from the device as well as flag the individual. The app developer can choose to employ a 3rd party library to detect if the device has been rooted or jailbroken, and refuse the user access to the app.

{% tabs %}
{% tab title="Swift" %}
```swift
// Generate a random encryption key

var key = Data(count: 64)

_ = key.withUnsafeMutableBytes { bytes in

  SecRandomCopyBytes(kSecRandomDefault, 64, bytes)

}

// Open the encrypted Realm file

let config = Realm.Configuration(encryptionKey: key)

do {

  let realm = try Realm(configuration: config)

  // Use the Realm as normal

  let dogs = realm.objects(Dog.self).filter("name contains 'Fido'")

} catch let error as NSError {

  // If the encryption key is wrong, `error` will say that it's an invalid database

  fatalError("Error opening realm: \(error)")

}
```
{% endtab %}

{% tab title="Java" %}
```
byte[] key = new byte[64];
```

```java

new SecureRandom().nextBytes(key);

RealmConfiguration config = new RealmConfiguration.Builder()

  .encryptionKey(key)

  .build();

Realm realm = Realm.getInstance(config);

Realm Object Server

Coupled with the Realm Database, the Realm Object Server is the other half of the coin of the Realm Platform which can also be configured to present a stout security posture for your mobile stack. To configure the data while in transit from the mobile to your backend or vice versa, you will need to add a valid certificate to your backend Realm Object Server. Before starting the server open the configuration file in `/etc/realm/configuration.yml` to enable HTTPS and disable HTTP. Here is a sample configuration:

proxy: 

https: 

enable: true 

listen_address: ‘192.168.0.2' 

certificate_path ‘/etc/realm/myCert.pem`

private_key_path ‘/etc/realm/myKey.pem`

http: 

enable: false
```
{% endtab %}
{% endtabs %}

Because this is for syncing with mobile devices you cannot use a self-signed certificate and because this certificate will also be used for the ROS dashboard you should make sure that you install your certificate in your web browser. Be sure to allow only read and execute permissions on the certificates, for instance:

```bash
sudo chmod 500 myCert.pem
```

For more details on how to generate and set-up these certificates see [Securing with Let's Encrypt](securing-with-lets-encrypt.md).  

Network security is the foundation of any security posture. You may have noticed that the listen\_address was filled in with an IP address in the above HTTPS proxy setting. This will tell the realm process to listen on a specific interface attached to the server rather than the default `::`or `0.0.0.0` which will listen on all interfaces. It is recommended that the interface you specify is placed in a DMZ subnet which is locked down and behind a network firewall.

Network security should also be applied on the host server. On an Ubuntu server you can do this by using ufw, such as:

```bash
sudo apt-get install ufw

sudo ufw enable

sudo ufw deny http

sudo ufw allow ssh

sudo ufw allow 9443/tcp

```

At this point you can further lock it down by setting up a default deny rule that will drop traffic that has not been specifically allowed. Be careful not to lock yourself out of your own server by blocking access! Take note of all known services and ports including statsd and ssh when setting up your linux firewall.

SSH is the most common method to access the command line of your server and should be treated with the utmost protection. It is recommended to use a public/private key pair to login, to change the default port, and to disable root access. As an example to do this on Ubuntu open /etc/ssh/sshd\_config and change the following parameters:

```bash
Port 2675

Protocol 2 

PermitRootLogin no

```

Then restart the ssh service. It is also recommended to lock down sudo with a secure auth service and explicitly control who can perform actions on the server with role based access control. There are many more server side recommendations for security such as disabling shared memory, enabling SELinux, further hardening of network settings around DNS and IP spoofing, and not least of which is enabling disk encryption. For Ubuntu, [eCryptfs](https://help.ubuntu.com/lts/serverguide/ecryptfs.html) is the recommendation.  

Your application stack and server exposure will determine how much security hardening you need. Adding an intrusion detection and prevention system as well as denial of service protection can be a costly task but will pay off in the unlikely event you are subject to one of these attack vectors. Feel free to contact Realm to discuss what your risk tolerance is and what level of security investment is needed.

Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

