# Run ROS in the Background

Once you are ready to go into production, you will want to daemonize your application project so that it runs in the background and automatically on server startup. We recommend using [pm2](https://github.com/Unitech/pm2) to maintain instances of Realm Object Server in production. PM2 is a battle tested process manager that allows your applications to stay alive forever and helps with mission critical administration during downtime.

## Install PM2

You can install pm2 in your machine using npm with:

```bash
npm install pm2 -g
```

## Start ROS

{% tabs %}
{% tab title="JavaScript" %}
```bash
pm2 start path/to/myserver/src/index.js
```

Now your ROS instance is daemonized, monitored and kept alive forever.

You can supply a name to your started instance with the `--name` option:

```bash
pm2 start path/to/myserver/src/index.js --name my-ros
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

## Stop ROS

```bash
pm2 stop my-ros
```

## Restart ROS

```bash
pm2 restart my-ros
```

## Delete ROS

When you are no longer interested in registering a daemonized and monitored instance you can use `delete`:

```text
pm2 delete my-ros
```

Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

