# Installation

Realm Object Server is a Node application that is distributed via npm.  To help get you started quickly, we have a few different installation options.  If you are already familiar with Node and NPM, we suggest using the [manual install via npm](./#manual-install-for-those-familiar-with-node-and-npm).  If not, we suggest using our [quick start install](./#quick-start-install-for-those-unfamiliar-with-node-and-npm) via our own curl script.  

## Supported Operating Systems {#supported-operating-systems}

We currently support the following operating systems:

* macOS 10+
* Ubuntu 16.04+
* RHEL 6+
* CentOS 7+

We plan to add native support for Windows. In the meantime, you can [**run the server for development purposes via Docker**](https://github.com/realm/realm-server-side-samples/tree/master/15-running-ros-with-docker)**.**

## Quick Start Install

{% hint style="info" %}
This will install a number of items like NVM, Node, NPM at a global level.  If you have existing installations, they may potentially be upgraded / downgraded.  If this is an issue, please use our [manual install ](./#manual-install)
{% endhint %}

The simplest method of installation is by using our install script, which will resolve all prerequisites for you:

```bash
curl -s https://raw.githubusercontent.com/realm/realm-object-server/master/install.sh | bash
```

## Manual Install

### Prerequisites {#prerequisites}

To get started, please ensure you have Node installed on your machine. Instructions are available [here](https://nodejs.org/en/download/package-manager/) and we recommend using [NVM](https://github.com/creationix/nvm).

{% tabs %}
{% tab title="Ubuntu" %}
```bash
// Ubuntu 16.04 (64 bit; 32-bit is not supported)
//It is recommended that you install the server as a normal user.

sudo apt-get update

sudo apt-get install build-essential libssl-dev

sudo apt-get install python

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.4/install.sh | bash

// Force current session to know changes or logout and log back in
source ~/.profile

nvm install --lts

npm install -g node-gyp
```
{% endtab %}

{% tab title="macOS" %}
```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.4/install.sh | bash

nvm install --lts

npm install -g node-gyp
```
{% endtab %}

{% tab title="RHEL/CentOS" %}
```text
//Install Node.js LTS
curl --silent --location https://rpm.nodesource.com/setup_8.x | sudo bash -
sudo yum -y install nodejs

//To compile and install native addons from npm
sudo yum install gcc-c++ make
```
{% endtab %}
{% endtabs %}

### Install the Server

```bash
npm install -g realm-object-server
```

This will install the server globally which is the easiest way to try it out since it includes a CLI. For Linux, if you are installing as root \(not recommended\), you may need to add `--unsafe-perm` to the `npm install` commands.

## Connecting to the Server

Connections to the server are made via websockets which communicate \(by default\) over port 9080, so you'll need to be sure to open this port.  The port number can be reconfigured within your server's index file.  The simplest way to [test connectivity is by using Realm Studio](../../realm-studio/).

## Upgrading the Server

Upgrading the server is as simple as running an NPM install command within your ROS project

```bash
#specify specific version after the @
npm install realm-object-server@latest
```

## Troubleshooting a failed installation

Having issues with your installation?  We're here to help.  Please [contact us](https://support.realm.io/) via our support channel.  If possible, include the following information: 

* Steps that you followed during the installation 
* Relevant error messages and/or logs
* Versions of NPM and Node
* Operating system information

## What's next?  [Learn how to start the server](../running-the-server.md) {#getting-started}

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3)

