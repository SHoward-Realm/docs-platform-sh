# Manual Install

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

## What's next?  [Learn how to start the server](../running-the-server.md) {#getting-started}

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

