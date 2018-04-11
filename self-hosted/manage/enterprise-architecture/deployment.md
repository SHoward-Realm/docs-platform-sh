# Deployment

The ROS components are designed to be deployed as node applications. The prerequisites for the server that any ROS service gets installed on can be found [here](https://realm.io/docs/realm-object-server/latest/#install-realm-object-server%20).

Once the prerequisites are installed the ROS services can be run as regular node applications as in executing the command: `node index.js` from the command-line**  
**

These commands and the installation of prerequisites are typically not done manually by an operator but instead is done with a configuration management tool like Chef, Puppet, Ansible, or Salt. These scripts are triggered on failure of a server which can be provided by a monitoring tool or for initial deployment which is usually manually started.  


Additionally, it is recommended to use a process monitor like pm2 to ensure that the process stays up or is restarted in the event of error or reboot, see [here](../run-ros-in-the-background.md) for instructions.  

## Starting the various Services

### Starting the Proxy and Auth Services

These two services are currently united under one command.

```bash
### set environment variables
CONSUL_HOST=<host>:<port>
FEATURE_TOKEN=<base64-encoded-and-signed> # provided by Realm
PRIVATE_KEY_PATH=<path>
PUBLIC_KEY_PATH=<path>
# don't forget to export all of those

### run the proxy and auth services
ros-enterprise run auth -p 9080 -a 0.0.0.0
```

### Starting the Replicated Sync Services

In order to start the Replicated Sync services, repeat this on all servers where you want to run them.

```bash
### set environment variables
CONSUL_HOST=<host>:<port>
FEATURE_TOKEN=<base64-encoded-and-signed> # provided by Realm
PRIVATE_KEY_PATH=<path> # use the same key pair
PUBLIC_KEY_PATH=<path> # for all your services
SYNC_ADDRESS=<host>
SYNC_PORT=<port>
SYNC_ID=<string>
SYNC_LABEL=<string>
# don't forget to export all of those

### run one of the replicated sync services
ros-enterprise run resync -p 9080 -a host1
```

`SYNC_ADDRESS` is used by the proxy and other workers to connect to the worker, so this cannot be `127.0.0.1` or `0.0.0.0`, use the real address at which the worker is to be found on the network.

`SYNC_LABEL` defines the sync worker group which serves as a single shard for load balancing. Should be `default` in the unbalanced case.

`SYNC_ID` uniquely identifies a sync worker among its worker group.

The services can use different values for `CONSUL_HOST`, if you run \(and you should\) multiple consul servers.



{% hint style="info" %}
_**WARNING**:_ A worker at a particular data directory should always be started with the same `SYNC_ID` and `SYNC_LABEL`, or else you **lose your data**.
{% endhint %}

{% hint style="info" %}
_**WARNING**:_ Always start at least three workers per label, because it needs at least two to successfully serve clients.
{% endhint %}

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

  


