---
description: How to setup disaster recovery
---

# Disaster Recovery

## Prerequisites: 

You'll need to install a second cluster just like your first cluster.  You can do this by following the instructions [here](deployment-instructions.md).  

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

And that’s it - your sync workers are ready to go. Simply startup the sync worker and other services when you are ready to failover and shift your global load balancer or DNS at the new disaster recovery public IP address. To make a seamless user experience you will want to add a client reset callback in the event that a DR event occurs. See [here](https://realm.io/docs/swift/latest/#client-reset) for more details. 

  


