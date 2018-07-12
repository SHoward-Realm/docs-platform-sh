# Command-line Interface for ROS

## General Usage

The Realm Object Server command-line utility provides additional functionality as well as a means to configure your server:

```javascript
$ ros --help
  Usage: ros [options] [command]


  Options:

    -V, --version  output the version number
    -h, --help     output usage information


  Commands:

    start [options]               Start Realm Object Server
    migrate [options]             Migrate data from Realm Object Server 1.x format to 2.0 format. Existing 1.x data is preserved.
    backup [options]              Create a backup of the Realm Object Server.
    init [options] <projectName>  Create a new ROS project

  Help for individual commands:

    ros init --help
    ros start --help
    ros migrate --help
    ros backup --help
```

## init

The `ros init` command will create a Javascript or Typescript project that includes Realm Object Server for you.

{% hint style="info" %}
When running a `ros init` you will receive the latest version of the Realm Object Server even if your ROS cli is an older version.  If you need to use a specific version of ROS, you can make these adjustments via NPM within your newly created project folder.  
{% endhint %}

{% tabs %}
{% tab title="TypeScript" %}
Let's say you want to create a project named `MyServer`. You can then run:

```bash
ros init MyServer
```

You'll now have a project directory structure that looks like this:

```text
MyServer
│   package.json
└───src
│   │   index.ts
```

With TypeScript 2.5.3, you'll have a few more scripts that you can run:

* `npm build` which will build all the `src/**/*.ts` files into equivalent `dist/**/*.js` files
* `npm clean` which will clean all the js artifacts from `dist/`
* `npm start` which will build the `.ts` files and run `node dist/index.js`

**To start MyServer:**

1. go into the working directory by `cd MyServer`
2. run `npm start`

To run this in a production enviroment see the ["Going Into Production" documentation](#going-into-production).

You can configure the server by editing the `index.ts` file which inclues comments on the available configuration options:

```typescript
import { BasicServer } from 'realm-object-server'
import * as path from 'path'

const server = new BasicServer()

server.start({
        // This is the location where ROS will store its runtime data
        dataPath: path.join(__dirname, '../data')

        // The address on which to listen for connections
        // Default: 0.0.0.0
        // address?: string

        // The port on which to listen for connections
        // Default: 9080
        // port?: number

        // Override the default list of authentication providers
        // authProviders?: IAuthProvider[]

        // Autogenerate public and private keys on startup
        // autoKeyGen?: boolean

        // Specify an alternative path to the private key. Otherwise, it is expected to be under the data path.
        // privateKeyPath?: string 

        // Specify an alternative path to the public key. Otherwise, it is expected to be under the data path.
        // publicKeyPath?: string

        // The desired logging threshold. Can be one of: all, trace, debug, detail, info, warn, error, fatal, off)
        // Default: info
        // logLevel?: string

        // Enable the HTTPS Server.
        // https?: boolean

        // The port on which to listen for HTTPS connections.
        // Default: 0.0.0.0
        // httpsAddress?: string

        // The address on which to listen for HTTPS connections.
        // Default: 9443
        // httpsPort?: number

        // The path to your HTTPS private key in PEM format. Required if HTTPS is enabled.
        // httpsKeyPath?: string

        // The path to your HTTPS certificate chain in PEM format. Required if HTTPS is enabled.
        // httpsCertChainPath?: string

        // Specify the length of time (in seconds) in which access tokens are valid.
        // Default: 600 (ten minutes)
        // accessTokenTtl?: number

        // Specify the length of time (in seconds) in which refresh tokens are valid.
        // Default: 3153600000 (ten years)
        // refreshTokenTtl?: number
        
        // Specify the largest size of changes considered for log compaction.
        // Default: 16000000 (16MB)
        // maxDownloadSize?: number
    })
    .then(() => {
        console.log(`Your server is started `, server.address)
    })
    .catch(err => {
        console.error(`There was an error starting your file`)
    })
```
{% endtab %}

{% tab title="Javascript" %}
If you prefer to use Javascript then you can create a JS Realm Object Server template with:

```bash
ros init MyServer --template js
```

You'll now have a project directory structure that looks like this:

```text
MyServer
│   package.json
└───src
│   │   index.js
```

To start `MyServer`:

1. go into the working directory by running `cd MyServer`
2. run `npm start`

To run this in a production enviroment see the ["Going Into Production" documentation](#going-into-production).

You can configure the server by editing the `index.js` file which inclues comments on the available configuration options:

```javascript
const BasicServer = require('realm-object-server').BasicServer
const path = require('path')
const server = new BasicServer()

server.start({
        // This is the location where ROS will store its runtime data
        dataPath: path.join(__dirname, '../data')

        // The address on which to listen for connections
        // Default: 0.0.0.0
        // address?: string

        // The port on which to listen for connections
        // Default: 9080
        // port?: number

        // Override the default list of authentication providers
        // authProviders?: IAuthProvider[]

        // Autogenerate public and private keys on startup
        // autoKeyGen?: boolean

        // Specify an alternative path to the private key. Otherwise, it is expected to be under the data path.
        // privateKeyPath?: string 

        // Specify an alternative path to the public key. Otherwise, it is expected to be under the data path.
        // publicKeyPath?: string

        // The desired logging threshold. Can be one of: all, trace, debug, detail, info, warn, error, fatal, off)
        // Default: info
        // logLevel?: string

        // Enable the HTTPS Server.
        // https?: boolean

        // The port on which to listen for HTTPS connections.
        // Default: 0.0.0.0
        // httpsAddress?: string

        // The address on which to listen for HTTPS connections.
        // Default: 9443
        // httpsPort?: number

        // The path to your HTTPS private key in PEM format. Required if HTTPS is enabled.
        // httpsKeyPath?: string

        // The path to your HTTPS certificate chain in PEM format. Required if HTTPS is enabled.
        // httpsCertChainPath?: string

        // Specify the length of time (in seconds) in which access tokens are valid.
        // Default: 600 (ten minutes)
        // accessTokenTtl?: number

        // Specify the length of time (in seconds) in which refresh tokens are valid.
        // Default: 3153600000 (ten years)
        // refreshTokenTtl?: number
        
        // Specify the largest size of changes considered for log compaction.
        // Default: 16000000 (16MB)
        // maxDownloadSize?: number
    })
    .then(() => {
        console.log(`Your server is started `, server.address)
    })
    .catch(err => {
        console.error(`There was an error starting your file`)
    })
```
{% endtab %}
{% endtabs %}

## start

{% hint style="warning" %}
This command is meant for demo purposes only
{% endhint %}

The `start` command will start the Realm Object Server and listen for connections. This mode is perfect for demoing or quickly starting the server. However, it will be tied to your terminal window and isn't recommended for production \(use `ros init`\). By default, no arguments are needed. However, if you would like to adjust some behavior of Realm Object Server, you can provide some configuration options:

| CLI Option | Default | Description |
| :--- | :--- | :--- |
| `--data <path>` | `./data` | This is the location where ROS will store its runtime data |
| `--address <address>` | `0.0.0.0` | The address on which to listen for connections |
| `--port <port>` | `9080` | The port on which to listen for connections |
| `--loglevel <level>` | `info` | The desired logging threshold. Can be one of: `all`, `trace`, `debug`, `detail`, `info`, `warn`, `error`, `fatal`, `off)` |
| `--logfile <path>` |  | Log to the specified file instead of the console |
| `--no-auto-keygen` |  | Do not autogenerate public and private keys on startup |
| `--private-key <path>` |  | Specify an alternative path to the private key. Otherwise, it is expected to be under the data path. |
| `--public-key <path>` |  | Specify an alternative path to the public key. Otherwise, it is expected to be under the data path. |
| `--refresh-token-ttl <seconds>` | `3153600000`\(ten years\) | Specify the length of time \(in seconds\) in which refresh tokens are valid. |
| `--access-token-ttl <seconds>` | `600` \(ten minutes\) | Specify the length of time \(in seconds\) in which access tokens are valid. |
| `--auth <provider1[,provider2 ...]>` | `password` | Override the default list of authentication providers. Multiple auth providers may be specified by separating them with a comma. |
| `--https` |  | Enable the HTTPS Server. |
| `--https-key <path>` |  | The path to your HTTPS private key in PEM format. Required if HTTPS is enabled. |
| `--https-cert <path>` |  | The path to your HTTPS certificate chain in PEM format. Required if HTTPS is enabled. |
| `--https-address <address>` | `0.0.0.0` | The address on which to listen for HTTPS connections. |
| `--https-port <port>` | `9443` | The port on which to listen for HTTPS connections. |



Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

