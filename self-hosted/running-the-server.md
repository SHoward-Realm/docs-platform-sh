# Running the Server

## Running the Server

Realm Object Server is a Node package which requires a Node project to run within. To make it easy to get going, the installation includes a command-line interface to bootstrap a project for you!

### Create a new ROS project instance:

```bash
ros init my-app
```

This creates a Typescript-based Node project for you. Later on you can explore customizing it but for now simply start the server with defaults by:

#### 1\) Add your feature token:

{% hint style="info" %}
Starting with Realm Object Server 3.5.0, a feature token is required to run the server. **You can** [**get your own feature token here**](https://realm.io/trial/self-hosted-standard-plan/)**.**
{% endhint %}

From inside of your project folder, you'll need to open the index file which can be found at `src/index.ts`You'll need to paste in your feature token as shown below:

```typescript
import { BasicServer, FileConsoleLogger } from 'realm-object-server'
import * as path from 'path'

const server = new BasicServer()

server.start({
        // ..
        featureToken: '<INSERT YOUR FEATURE TOKEN HERE>',
        //....
    })
```

#### 2\) Start the server

```bash
cd my-app/
npm start
```

That’s it! You now have a functioning Realm Object Server running locally on port 9080! The server is tied to your terminal window for now and can be stopped by pressing Ctrl-C.

{% hint style="info" %}
If you have trouble running a `ros` command it may be due to your version of `NVM`. Try running `nvm use 8` in your terminal.
{% endhint %}

Once you are ready to go into production with the server you will want to run it in the background. See the documentation section ["Going Into Production"](running-the-server.md) for more details.

{% hint style="info" %}
Click [here](manage/command-line-interface-for-ros.md) for more information on the command-line interface for ROS.
{% endhint %}

## Running ROS in the Background

Once you are ready to go into production, you will want to daemonize your application project so that it runs in the background and automatically on server startup. We recommend using [pm2](https://github.com/Unitech/pm2) to maintain instances of Realm Object Server in production. PM2 is a battle tested process manager that allows your applications to stay alive forever and helps with mission critical administration during downtime.

### Install PM2

You can install pm2 in your machine using npm with:

```bash
npm install pm2 -g
```

### Start ROS

{% tabs %}
{% tab title="JavaScript" %}
```bash
pm2 start path/to/myserver/dist/index.js
```

Now your ROS instance is daemonized, monitored and kept alive forever.

You can supply a name to your started instance with the `--name` option:

```bash
pm2 start path/to/myserver/dist/index.js --name my-ros
```
{% endtab %}

{% tab title="TypeScript" %}
If you’re interested in starting the TypeScript source you’ll need to have pm2 install TypeScript via:

```bash
pm2 install typescript
```

You will only have to run the install once.

Then run.

```bash
pm2 start my-ros/src/index.ts
```

You can supply a name to your started instance with the `--name` option:

```bash
pm2 start path/to/myserver/dist/index.ts --name my-ros
```
{% endtab %}
{% endtabs %}

### Stop ROS

```bash
pm2 stop my-ros
```

### Restart ROS

```bash
pm2 restart my-ros
```

### Delete ROS

When you are no longer interested in registering a daemonized and monitored instance you can use `delete`:

```text
pm2 delete my-ros
```

## What's next - [Install Realm Studio](../realm-studio/#installation)

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3)

