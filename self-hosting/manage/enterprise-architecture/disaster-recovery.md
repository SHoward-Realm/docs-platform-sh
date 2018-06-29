# Disaster Recovery

This guide will go over the process to setup disaster recovery for your Realm Object Server.  To learn about how disaster recovery works at a high level, click [here](high-availability.md#disaster-recovery).  

## Prerequisites: 

You'll need to install a second cluster just like your first cluster.  You can do this by following the instructions [here](manual-deployment-instructions.md).  

{% hint style="danger" %}
Do not start the cluster as you might trigger a client reset when you actually need to recover.
{% endhint %}

You'll want to do this in a different region to help mitigate risk.  In this example, I have spun up a cluster in the AWS West whereas my live cluster that is serving traffic is in AWS East. You will need a matching sync worker group in DR for each sync worker group in your live production cluster.

## Preparing your server

Be sure to copy the `auth.key` and `auth.pub` from the primary site to the DR site. The backup command does bring the keys to the remote site along with the data but you may have auxiliary services like CoreServices which do not need a backup because they are built off the sync-workers. Also, be sure to match the `SYNC_LABEL` of the primary site for each sync-worker group in the DR site.

Go to your Primary Cluster and SSH to the master sync worker and navigate to the data directory in the sync worker folder. You can find out the master quickly by going to the Consul UI &gt; Key/Value and then clicking on sync-worker-group/&lt;SYNC\_LABEL&gt;/master

![Consul Admin UI](https://lh3.googleusercontent.com/1snstS5zmUxzqQu5_GWJtw5PTtHKUQ0UvmQ9BGoZ4LAyxrG5dBP4_0ZwxDChjJCak0hfx524MwJfNVsu7To97N0ZkL9bSebGyGInqb9Ncmqf2nFGxF0hitVmo8SqejoCvn_kyNWb)

Looking at the UI, you can find the master swg: 

`{"master":{"id":"swg-01-east","address":"10.237.244.116","port":7800},"slave":{"complete":true,"id":"swg-03-east"}}`  


You can see that swg-01-eastis the master. SSH to that node.

In the data directory you will need to `symlink` user\_data to a new directory called sync by using a command like this:

```bash
ubuntu@swg-01-east:~/syncWorker/data$ ln -s user_data/ sync
```

If you execute a `ls -la` command in the data folder you should see:

```text
sync -> user_data/
```

## Creating a backup

Go back to the syncWorker folder and create a folder called backup, the run the following command:

```bash
ubuntu@swg-01-east:~/syncWorker$ node_modules/realm-object-server/dist/cli.js backup -f data/ -t backup
```

Then you should see something like the following:

```text
info: The data directory /home/ubuntu/syncWorker/data was backed up in the directory /home/ubuntu/syncWorker/backup.
11 Realms were successfully backed up
```

If you go into the backup folder you should see a `keys/` folder and a `sync/` folder. 

## Copying the backup to the remote master

Now we need a way of copying our backup folder to the remote master syncWorker in the DR cluster. You can use many tools to move this folder to the remote host such as distributed file-system, storage. In this example I will use rsync and an SSH key.  

```bash
#SSH key that maps to a corresponding key-pair on the remote host
ubuntu@swg-01-east:~/syncWorker$ vi myKey.pem
#Make the permissions secure
ubuntu@swg-01-east:~/syncWorker$ sudo chmod 400 myKey.pem
#Use rsync and your SSH key to transfer it over
ubuntu@swg-01-east:~/syncWorker$ rsync -avz -e "ssh -i myKey.pem" backup/ ubuntu@<IP_OF_REMOTE_SYNC_WORKER>:remoteBackup
```

You will need to run the rsync command for each sync worker in your group - typically this is three. SSH to each remote sync worker node and rename that sync folder to `user_data`  


```bash
ubuntu@swg-01-west:~/remoteBackup$ mv sync/ user_data
#Now copy the backup folder to the sync worker data directory
ubuntu@swg-01-west:~$ cp -a remoteBackup/. syncWorker/data/
```

And that’s it - your sync workers are ready to go. Simply startup the sync worker and other services when you are ready to failover and shift your global load balancer or DNS at the new disaster recovery public IP address. 

All of the above manual steps can be put into a Bash script that can be executed on a regular basis via a cron job. For instance, below the script grabs the IP address of the host server, checks to see if it is the master, and if it is creates a backup and copies it to the remote sync-worker cluster.

```bash
#!/usr/bin/sh -eo pipefail
machineIp=$(ip addr show eth0 | grep -Eo '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | head -1)
echo $machineIp
consul_host="<CONSUL_HOST>"
consul_response=$(curl -s "http://$consul_host:8500/v1/catalog/service/sync?tag=role=master")
service_address_regex="\"ServiceAddress\":\\s*\"([^\"]*)\""
if [[ $consul_response =~ $service_address_regex ]]; then
   sync_master="${BASH_REMATCH[1]}"
   echo $sync_master
   if [[ $sync_master = $machineIp ]]; then
     echo "I am the master sync-worker : " $sync_master
    cd ~/syncworker
    rm -rf backup
    mkdir -p backup
    node_modules/realm-object-server/dist/cli.js backup -f data -t backup
    rsync -avz -e "ssh -i myKey.pem" backup/ ubuntu@<IP_OF_REMOTE_SYNC_WORKER>:syncworker/data/
    rsync -avz -e "ssh -i myKey.pem" backup/ ubuntu@<IP_OF_REMOTE_SYNC_WORKER>:syncworker/data/
    rsync -avz -e "ssh -i myKey.pem" backup/ ubuntu@<IP_OF_REMOTE_SYNC_WORKER>:syncworker/data/
   fi
else
   echo "Could not determine the sync master. Consul response follows:"
   echo $consul_response
fi
```

To make a seamless user experience you will want to add a client reset callback in the event that a DR event occurs. See [here](https://realm.io/docs/swift/latest/#client-reset) for more details. 

Below is a node script you can run that accomplishes the same effect but includes fault tolerance.

```javascript
#!/usr/bin/env node

const configuration = {
    /** Path to the sync worker on this and the destination servers */
    syncWorkerPath: "/apps/syncworker",
    consulHost: "realm-ecc.nndc.kp.org",
    /** Network interface that the sync worker is listening on. Used to compare sync services addresses. */
    networkInterface: "eth0",
    destination: {
        servers: [ 
            "cskpcloudxp3258.cloud.kp.org",
            "cskpcloudxp3259.cloud.kp.org",
            "cskpcloudxp3260.cloud.kp.org"
        ],
        /** SSH username for the remote servers */
        username: "ecc",
        /** SSH keyfile to authenticate as `username` with */
        pathToKeyfile: `${__dirname}/keyfile.pem`
    },
    /** How long do we wait for a single rsync job before we kill it and bail */
    rsyncTimeout: 30 * 60 * 1000, // 30 minutes, in milliseconds,
    emailErrorHandling: {
        /** Who to send error messages to. */
        mailbox: "info@example.tld",
        /** Options for `nodemailer.createTransport`. See https://nodemailer.com/smtp */
        nodemailerTransportOptions: undefined
    },
}

const { promisify } = require("util");
const { networkInterfaces, hostname } = require("os");
const exec = promisify(require("child_process").exec);

const Rsync = require("rsync");
const Consul = require("consul");
const nodemailer = require("nodemailer");
const mkdirp = promisify(require("mkdirp"));
const rimraf = promisify(require("rimraf"));
const chmodr = promisify(require("chmodr"));

async function getSyncMasterAddress() {
    const consul = new Consul({
        host: configuration.consulHost,
        promisify: true
    });

    const services = await consul.catalog.service.nodes({ service: "sync", tag: "role=master" });
    if (services.length !== 1) {
        throw new Error(`Expected exactly one sync master in Consul but got ${services.length}`);
    }
    return services[0].ServiceAddress;
}

async function runRealmBackup() {
    await rimraf(`${configuration.syncWorkerPath}/backup`); // rm -rf
    await mkdirp(`${configuration.syncWorkerPath}/backup`); // mkdir -p

    console.log('Running ROS backup command.');
    await exec('node_modules/.bin/ros backup -f data -t backup', {
        cwd: configuration.syncWorkerPath,
        env: {
            "ROS_SKIP_PROMPTS": "1",
            ...process.env
        }
    });
    await chmodr(`${config.syncWorkerPath}/backup`, 0777); // chmod -R
}

async function runRsync(server) {
    const rsync = new Rsync()
        .shell(`ssh -i ${configuration.destination.pathToKeyfile}`)
        .source(`${configuration.syncWorkerPath}/backup/`)
        .destination(`${configuration.destination.username}@${server}${configuration.syncWorkerPath}`)
        .archive()
        .compress()
        .set("omit-dir-times");
    
    try {
        await exec(rsync.command(), { timeout: configuration.rsyncTimeout });
    } catch (e) {
        if (e.code === null && e.signal === "SIGTERM") {
            throw new Error(`rsync to ${server} timed out.`);
            console.error(e);
        } else {
            throw e;
        }
    }
}

async function main() {
    console.log(`Starting backup job at ${Date()}`);

    const currentSyncMasterAddress = await getSyncMasterAddress();
    console.log(`${currentSyncMasterAddress} is sync master.`);

    const localAddress = networkInterfaces()[configuration.networkInterface].filter(i => i.family === "IPv4")[0].address;
    if (localAddress !== currentSyncMasterAddress) {
        console.log('I am not sync master. Exiting.');
        return;
    }

    console.log('I am sync master. Proceeding with backup.');
    await runRealmBackup();

    for (let server of configuration.destination.servers) {
        console.log(`Running rsync to ${server}`);
        await runRsync(server);
    }

    console.log(`Backup done at ${Date()}. Cleaning up.`);
    await rimraf(`${configuration.syncWorkerPath}/backup`); // rm -rf
}

async function sendErrorEmail(error) {
    const transport = nodemailer.createTransport(configuration.emailErrorHandling.nodemailerTransportOptions);

    await transport.sendMail({
        from: `realm-backup-script@${hostname()}`,
        to: configuration.emailErrorHandling.mailbox,
        subject: "Error",
        text: JSON.stringify(error)
    });
}

main().catch(async error => {
    console.error("Backup failed.");
    console.error(error);
    
    if (configuration.emailErrorHandling.nodemailerTransportOptions) {
        await sendErrorEmail(error);
    }

    process.exit(1);
});
```

### Creating a cronjob

You can set up a cronjob through the use of crontab with the command:

```text
env EDITOR=nano crontab -e
```

In the text editor you can edit the frequency at which the following commands will be executed. You can read more about the syntax [here](https://code.tutsplus.com/tutorials/scheduling-tasks-with-cron-jobs--net-8800).

An example cronjob may look like this:

```text
30 * * * *  cd ~/Desktop && ./script.sh
```

This creates a command to run hourly at the 30th minute \(1:30, 2:30, etc.\) by changing to the Desktop directory and running the desired script.

  
Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

