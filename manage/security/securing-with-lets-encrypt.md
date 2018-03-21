# Securing with Let's Encrypt

So, you've developed an app with the [Realm Platform](https://realm.io/products/realm-mobile-platform/), and you're ready to go live. Before you hit the publish button though, consider the fact that it's the 21st century---your users expect that their data is secure and not being snooped on. Imagine how much a privacy scandal would hurt the adoption of your app! The good news is that it takes only 10--15 minutes to setup proper encryption.

[Let's Encrypt](https://letsencrypt.org) is free and automated---no need to deal with email reminders, copying certificates, or paying fees. What's not to like?

### Before we begin {#before-we-begin}

These instructions apply to setting up a Realm Object Server on a Ubuntu instance. For self-sufficiency's sake, I'll include all commands needed to make it work there, but will also link to the official docs if you have a different setup.

To obtain a certificate, you'll need to have a domain name registered. If you're hosting your server on a public cloud, Let's Encrypt won't let you register a certificate for a DNS with the cloud provider's top-level domain \(e.g., `myserver.something.azure.com`\).

Finally, this guide will assume you have a very basic understanding of Linux. If you know what `ls` does and [how to quit vim](https://stackoverflow.com/questions/11828270/how-to-exit-the-vim-editor), you're good to go. \(If you know how to recompile the kernel, then I'm afraid you'll get bored with these instructions rather quickly.\)

### Installing the Realm Object Server {#installing-the-realm-object-server}

If your server is hosted in a cloud environment, you'll probably need to open ports `9080` and `9443`; those are used by Realm for `http` and `https` traffic respectively. Additionally, you'll need to open `443`, as it will be used by Let's Encrypt to verify domain ownership. If you're self-hosting, you may need to open them in the built-in firewall:

```bash
sudo ufw allow 9080/tcp
sudo ufw allow 9443/tcp
sudo ufw allow 443/tcp
sudo ufw reload
```

Instructions for other Linux distributions can be found under [Troubleshooting](../../troubleshoot/verify-port-access.md)

After this, setting up ROS is fairly straightforward.  You can find the instructions [here](../../getting-started/install-realm-object-server/).  

You should now be able to connect to your ROS instance via Realm Studio to create an admin user.  

Now that the server is installed, let's configure it---fire up your favorite editor \([which clearly is vim](http://www.viemu.com/a-why-vi-vim.html)\) and edit `/src/index.ts`. There are a bunch of useful configuration options there, but what we care about is the `proxy:https` section, which is roughly in the middle of the file. Let's uncomment the settings and set the certificate paths.  The `https` section of your `index.ts` will look roughly like the following:

```typescript
        // Enable the HTTPS Server.
        // https?: boolean = false
        https: true,

        // The port on which to listen for HTTPS connections.
        // httpsAddress?: string = '0.0.0.0',
        // httpsAddress: '0.0.0.0',

        // The address on which to listen for HTTPS connections.
        // httpsPort?: number = 9443
        // httpsPort: 9443,

        // The path to your HTTPS private key in PEM format. Required if HTTPS is enabled.
        // httpsKeyPath?: string
        httpsKeyPath: '/etc/realm/keys/privkey.pem',

        // The path to your HTTPS certificate chain in PEM format. Required if HTTPS is enabled.
        // httpsCertChainPath?: string
        httpsCertChainPath: '/etc/realm/keys/fullchain.pem',
```

If you restart the server now with `npm start`, you'll notice that it fails to start due to configuration errors---the certificate paths are invalid. We'll fix them in a bit.

### Obtaining certificates {#obtaining-certificates}

There are a [gazillion ACME clients](https://letsencrypt.org/docs/client-options/) for Let's Encrypt out there, but we'll use the recommended one, [Certbot](https://certbot.eff.org/). Let's install it and fire it up:

```bash
# install certbot
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install certbot

# fire it up in certonly mode
sudo certbot certonly
```

\(Instructions for other Linux flavors can be found on the [Certbot website](https://certbot.eff.org).\)

Now, you should be presented with a question---do you want to spin up a temporary server, or use webroot? Unless you already have a web server running on that box, you should probably choose the former.

Next you'll be asked for domain name\(s\). Since Let's Encrypt won't issue wildcard certificates and you probably don't want to host your website on that box, you can setup a DNS record for something like `ros.myserver.com` and point it to your server. If everything goes well, certbot will negotiate with Let's Encrypt and place your certificates in `/etc/letsencrypt/live/ros.myserver.com/`.

### Providing the certificates to the Object Server {#providing-the-certificates-to-the-object-server}

Now we have ROS and our certificates, but there are a few more things to do before being able to relax and enjoy our secure communication. By default, certbot will apply severe restrictions on who can read the certificates folder---and that's a good thing---but those restrictions will prevent ROS from accessing that folder, too. We don't want to mess with certbot's permissions structure, so instead we'll copy relevant certificates to the `/etc/realm/keys` folder.

First, let's create the directory:

```bash
sudo mkdir /etc/realm/keys
sudo chown realm:realm /etc/realm/keys
sudo chmod 500 /etc/realm/keys
```

This creates the folder, changes the owner to `realm` \(the user under which ROS runs\), and sets the permissions to read and execute so ROS can [access files within](https://unix.stackexchange.com/questions/21251/execute-vs-read-bit-how-do-directory-permissions-in-linux-work/21252#21252).

At this point we could copy the files manually inside the folder, but that's not future-proofed---Let's Encrypt certificates are relatively short-lived \(90 days\), so you'd need to not only renew sooner rather than later, you'd have to remember to copy the files, which is error-prone. Instead, we'll prepare a [certbot renew hook script](https://certbot.eff.org/docs/using.html#renewing-certificates) that will be executed after successful renewal:

```text
#!/bin/sh

set -e

for domain in $RENEWED_DOMAINS; do
        case $domain in
        ros.myserver.com) # <- Update this with your own server
                realm_cert_root=/etc/realm/keys

                # Make sure the certificate and private key files are
                # never world readable, even for an instant, while
                # we're copying them into realm_cert_root.
                umask 077

                cp "$RENEWED_LINEAGE/fullchain.pem" "$realm_cert_root/fullchain.pem"
                cp "$RENEWED_LINEAGE/privkey.pem" "$realm_cert_root/privkey.pem"

                # Apply the proper file ownership and permissions for
                # Realm to read its certificate and key.
                chown realm:realm "$realm_cert_root/fullchain.pem" \
                        "$realm_cert_root/privkey.pem"
                chmod 400 "$realm_cert_root/fullchain.pem" \
                        "$realm_cert_root/privkey.pem"

                # restart ROS
                systemctl restart realm-object-server
                ;;
        esac
done
```

Here's the human-readable explanation of the various steps:

1. Check that we're copying the correct certificate \(`ros.myserver.com`\) and avoid touching unrelated certificates.
2. Copy the new certificates to `/etc/realm/keys`.
3. Set the owner of the copied file to the `realm` user.
4. Set permissions to "read" just for the file owner, and "none" for everyone else.
5. Restart ROS to make sure it picks up the new certificate.

Drop that in a non-public folder \(e.g., `/etc/realm/keys/`\) and make sure to make it executable \(`chmod 500 /etc/realm/keys/post-renew.sh`\).

Now, it's time to make sure everything works as expected:

```bash
sudo certbot renew --renew-hook /etc/realm/keys/post-renew.sh --force-renewal
```

This will force renewal and, upon success, execution of the renewal script. If everything goes according to plan, we should see the new certificates in the `/etc/realm/keys` folder, and restarting the ROS service should be able to pick them up. Head over to the browser and verify that you can open `https://ros.myserver.com:9443`.

The final step is to automate the renewal so we don't have to worry about the certificate expiring. Since we've verified that our `post-renew`script works as expected, we can safely setup a cron job to run the renew command:

```bash
sudo crontab -e

# choose your favourite editor and paste this at the bottom of the file
0 4 * * * certbot renew --renew-hook /etc/realm/keys/post-renew.sh --quiet
```

Certbot will only renew certificates nearing expiration, so running it daily is perfectly fine.

### Finale {#finale}

That's it. We now have a secured Realm Object Server instance that will renew and pick up new certificates without user interaction. It's a one-time setup that is definitely worth the effort to ensure that your users' data is not tampered with along the wire.

Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

