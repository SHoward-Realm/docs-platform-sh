# Hardware Requirements and Scaling

## Enterprise Production Deployment

The Realm process itself is single-threaded therefore only a single core is ever needed for ROS specific services. The size of the nodes varies by use-case but the  ROS Core Services and ROS sync-workers are designed to serve 10,000 concurrent connections from a single CPU server with 16GBs of RAM. The Realm files are files on disk and the sizing will be completely dependent on your dataset - a good starting point is to have 40GB of free disk space on the volume for Realm to have access to.  


Guidance of the sizing of Consul servers can be found [here](https://www.consul.io/docs/guides/performance.html).  Typically these are much smaller servers with only a single CPU a couple GBs of RAM needed.  


The Consul and Realm components can be deployed on the same server - if this is the case you will want to make sure that each process has its own CPU.

More details around the structure of the deployment can be found [here](./).  

## Development and Test Deployments

For development and test environments the hardware requirements can be drastically scaled down to only a few GBs of RAM needed with a single core. Dev environments are commonly installed on a local laptop either manually or with Docker as seen [here](https://docs.realm.io/platform/getting-started/install-realm-object-server/manual-install%20)

## Dependencies 

The Realm Components must be installed on a Linux server such as Ubuntu 16.04, there are a few dependencies that should be installed first such as Python, node.js, and npm - the exact details can be found [here](https://docs.realm.io/platform/self-hosted/installation).  

## Scaling out ROS components 

In the event of a ROS Core services node is overloaded as shown by a network monitoring tool multiple ROS Core services can be added to the pool by deploying more nodes using a CM tool and then added to the public load balancer configuration as healthy nodes in the LB pool.  


In the event of a ROS sync-worker group is overloaded as shown by a network monitoring tool another group can be added to the pool by deploying more nodes using a CM tool and by choosing a unique name for the `SYNC_LABEL` \(the same for every node in the group but different from other groups\) and a unique `SYNC_ID` for each sync-worker node. The group will automatically be discovered via consul and then new connections will start to take advantage of the increased capacity as distributed by the ROS proxy module.  


Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

