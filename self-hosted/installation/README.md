# Installation

## Supported Operating Systems {#supported-operating-systems}

We currently support the following operating systems:

* macOS 10+
* Ubuntu 16.04+
* RHEL 6+
* CentOS 7+

We plan to add native support for Windows. In the meantime, you may run the server for development purposes only via [Docker](https://docs.realm.io/platform/~/revisions/-L7Ry-Hl-dUJQWPWE8Nx/getting-started/install-realm-object-server/installing-via-docker).

## Quick Start Install (For those unfamiliar with Node and NPM)

The simplest method of installation is by using our install script, which will resolve all prerequisites for you:

```bash
curl -s https://raw.githubusercontent.com/realm/realm-object-server/master/install.sh | bash
```

## Manual Install (For those familiar with Node and NPM)

Realm Object Server is a Node application that is distributed via npm. It’s a prerequisite that Node.js \(6 or later\) is installed on your system. If you don’t have that, click below to see how to do that.

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


Upgrading the server is as simple as running an NPM install command within your ROS project 

```bash
#specify specific version after the @
npm install realm-object-server@latest
```

## What's next?  [Learn how to start the server](../running-the-server.md) {#getting-started}

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3)

