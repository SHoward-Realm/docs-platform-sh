# Running the Server

## Running the Server

Realm Object Server is a Node package which requires a Node project to run within. To make it easy to get going, the installation includes a command-line interface to bootstrap a project for you!

Simply run:

```bash
ros init my-app
```

This creates a Typescript-based Node project for you. Later on you can explore customizing it but for now simply start the server with defaults by:

```bash
cd my-app/
npm start
```

That’s it! You now have a functioning Realm Object Server running locally on port 9080! The server is tied to your terminal window for now and can be stopped by pressing Ctrl-C.

{% hint style="info" %}
If you have trouble running a `ros` command it may be due to your version of `NVM`.  Try running `nvm use 8` in your terminal.
{% endhint %}

Once you are ready to go into production with the server you will want to run it in the background. See the documentation section ["Going Into Production"](https://github.com/realm/docs-platform/tree/b245f7b94add1ca8ff81430ea06b466747799ffb/manage/run-ros-in-the-background.md) for more details.

{% hint style="info" %}
Click [here](https://docs.realm.io/platform/going-into-production/going-into-production/command-line-interface-for-ros) for more information on the command-line interface for ROS.
{% endhint %}

## What's next - [Install Realm Studio](../../realm-studio/#installation)

Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

