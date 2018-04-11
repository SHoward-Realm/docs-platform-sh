# Enterprise Architecture

## Key Components

The Realm Object Server Enterprise Edition allows for the component services of the Realm Object Server to be split out and scaled independently enabling elasticity with scaleout and high availability with failover.  In this section, we will summarize these core services and components which are involved in a typical enterprise deployment.   


![Example Deployment Diagram](https://lh3.googleusercontent.com/HAb4fdUC1szmnsngm_K83rfViwHiLGJXqBBDaFb2sZN4Aqt7DkTOyAvrnAy8HOIL0cgiX26A0p-lTRKJQf6Z4xTvrcRNzdaE2h3H38pQqGPjKZVV-pd4HaY3OGPiRnGZchOAyNS9)

### Realm Core Services

Realm Core Services is a stateless node application that contains many of the Realm administrative functions such as the proxy, authentication, and directory service. These services can be deployed independently or all in the same process by editing the services: array of the ServerStart config. Because it is stateless and independent it can be deployed on Bluemix or a similar PaaS solution. There can only be a single RealmDirectory service running at any time but all other services can be run in parallel for load balancing purposes. 

### Realm Sync Workers

Realm Sync Workers is a stateful node application that stores the realms, serves the data to sync clients, and merges changes. These are deployed in clusters of three with a master, backup, and spare - the master election is determined by consul which also detects failure and triggers failover. The sync worker group is configured with synchronous backup wherein each change written to the master is also written to the slave sync-worker before an acknowledgement is sent back the realm enabled mobile client. 

### Consul Cluster

Consul Cluster is a cluster of at least three consul servers that acts as a distributed key-value store for consensus and service discovery in the ROS cluster. It has built-in failure detection which the ROS cluster continually checks. If failure is detected the ROS Core services proxy module shifts the traffic to the new sync-worker master \(the old slave\) and the spare becomes the new slave. In small deployments the consul process can be run on the same server as the sync-workers and core services as shown below:**  
  
**

![](https://lh4.googleusercontent.com/HUc9FY3RD1ntJKpDrw_UUntEQHJITH-yaXTByNZZ4quasu9ej7BR1e928aubRfzz9vEIkerqx31IWms90LBy9XDddOCSMCXPQdJkbEqwdKae_qOa-QfdBwAP2OWxDdBBX-2Ew6yV)

However in larger deployments like with 10 or more ROS nodes it is recommended to split out the consul cluster into its own standalone cluster of at least three separate servers. 

### Public Load Balancer

This is typically a geo-distributed load balancer that is designed to forward any Realm traffic from remote connections to the Realm proxy. It must have a publicly accessible address.  It is commonly deployed in a pool using a redundancy protocol like VRRP and must be automated and programmable in the case that the DNS or IP of the Realm Core Services proxy module changes or fails over.

## Cluster Deployment

The following diagram shows the component pieces of the Realm Object Server Enterprise Edition deployed across a cluster. This also reiterates which of the components services are stateful vs stateless.  Deployment mediums should be considered based on these properties.  **  
  
**

![](https://lh5.googleusercontent.com/6HiFsj46qLrI76UdoOrnL54gcYNu2WYZcU51gsndb-ToGQ04uCZIkyPrblzPRuJBzfl7f8FYm3VQoCvVaaLWPuojJLCnUcXc9TCJIcRJp4CxMF-ykqQoHmcNxmGWgB3ufRj5_Qns)

  
Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

