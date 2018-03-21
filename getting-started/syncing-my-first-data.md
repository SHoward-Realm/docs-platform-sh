# Syncing My First Data

## Objective: 

Use `Node.js `and `realm-js` to load a sample dataset into the Realm Object Server and view the data through Realm Studio

## Installing Realm Studio

Realm Studio is Realm’s GUI console utility for managing and inspecting both local and synchronized Realms. In addition, you can connect to a running Realm Object Server to view logs and manage users.

![](https://s3.amazonaws.com/static.realm.io/docs/ros/images/getting-started-1.png)

### Download

**To get started with Realm Studio download the latest release:**

[Download for Mac](https://studio-releases.realm.io/latest/download/mac-dmg)

[Download for Linux](https://studio-releases.realm.io/latest/download/linux-appimage)

[Download for Windows](https://studio-releases.realm.io/latest/download/win-setup)

## Loading a Sample Data Set

Since there is no data yet in the server, let’s use `realm-js` to create a sample data set.

{% hint style="info" %}
**To complete this step, you will need to sign up for our Professional Edition trial **[**here**](https://realm.io/trial/realm-professional-edition/).

A token will be emailed to you that you can use in the sample application.
{% endhint %}

On your computer, create a new directory and run:

```bash
npm init

// This will ask you for info on your Node app, 
// just press Return until it creates a `package.json` for you
```

Next, let’s install the dependencies for you sample app:

```bash
// Install Realm 2.0 and save to package.json
npm install realm -S

// Install Faker library used by script and save to package.json
npm install faker -S
```

Create a file called `index.js` and paste in the following code:

```javascript
const faker = require('faker')
const Realm = require('realm')
const fs = require('fs')

var totalTickers = 100
var username = "realm-admin"
var password = ""
var URL = "127.0.0.1:9080"

const TickerSchema = {
  name: 'Ticker',
  primaryKey: 'tickerSymbol',
  properties: {
    tickerSymbol: { type: 'string', optional: false },
    price: { type: 'int', optional: false },
    companyName: { type: 'string', optional: false },
  }
}

const token = "INSERT PROFESSIONAL OR ENTERPRISE FEATURE TOKEN"; 
// Unlock Professional Edition APIs
Realm.Sync.setFeatureToken(token);

function generateRandomTickerSymbol(len) {
  charSet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
  let randomString = '';
  for (let i = 0; i < len; i++) {
    let randomPoz = Math.floor(Math.random() * charSet.length);
    randomString += charSet.substring(randomPoz, randomPoz + 1);
  }
  return randomString;
}

function getStarted() {
    Realm.Sync.User.login(`http://${URL}`, username, password)
    .then(user => {
        Realm.open({
            sync: {
                url: `realm://${URL}/tickers`,
                user: user
            },
            schema: [TickerSchema],
        })
        .then(realm => {
            let tickerResults = realm.objects('Ticker')
            // Add some data to the tickers Realms
            if (tickerResults.length < totalTickers) {
                realm.write(() => {
                    for (let index = 0; index < totalTickers; index++) {
                        realm.create('Ticker', {
                            tickerSymbol: generateRandomTickerSymbol(3),
                            price: index,
                            companyName: faker.company.companyName()
                        }, true)
                    }
                })
            }

            tickerResults.addListener((objects, changes) => {
                changes.modifications.forEach((index) => {
                    var ticker = objects[index];
                    console.log(`Ticker ${ticker.companyName} - ${ticker.tickerSymbol} - ${ticker.price}`)
                });
            })
        })
    })
}

getStarted();
```

This sample application will login to your ROS running locally via the default admin user and then create 100 Ticker objects in a synced Realm at path: `/tickers`. Just run the application via:

```bash
node index.js
```

You will now see `/tickers` in Realm Studio.  More on Realm Studio [here](https://docs.realm.io/platform/getting-started/administering-the-server#using-realm-studio).    

  


![](https://s3.amazonaws.com/static.realm.io/docs/ros/images/getting-started-5.png)

Click on the `/tickers` row to open the browser:  


![](https://s3.amazonaws.com/static.realm.io/docs/ros/images/getting-started-6.png)

The script also registers a listener to react to changes, so try editing the `companyName` property on one of the Ticker objects, you will see something like this print in your console:  


![](https://s3.amazonaws.com/static.realm.io/docs/ros/images/getting-started-7.png)

  
_**Congrats you have successfully synced your first data with Realm Object Server 2.0!**_

## Using Partial Synchronization

Part of Realm Platform 2.0, we are launching a preview of partial synchronization. This is a new and exciting capability where client applications can dynamically request data from Realm Object Server by registering queries. This is a major change in how application developers can use Realm in their apps. Previously, applications would need to segment their data into multiple Realms to control which data is synced at a given time. Now this can be done entirely in a single Realm!

Add this function to your code:

```javascript
function partialSyncTickers(greaterThanPrice) {
    Realm.Sync.User.login(`http://${URL}`, username, password)
    .then(user => {
        Realm.open({
            sync: {
                url: `realm://${URL}/tickers`,
                user: user,
                partial: true
            },
            schema: [TickerSchema],
        })
        .then(realm => {
            console.log(`Adding query to partially sync all Tickers greater than ${greaterThanPrice}`)

            realm.subscribeToObjects('Ticker', `price > ${greaterThanPrice}`)
            .then((results, error) => {
                console.log(`Partially synced ${results.length} Tickers`)

                results.addListener((results, changes) => {
                    if (changes.insertions.length > 0) {
                        console.log(`Partially synced ${results.length} Tickers`)
                    }
                    changes.modifications.forEach((index) => {
                        var ticker = results[index];
                        console.log(`Updated Partial Synced Ticker ${ticker.companyName} - ${ticker.tickerSymbol} - ${ticker.price}`)
                    })
                })
            })
        })
    })
}
```

Let’s walk through what this function does, step by step:

1. Like the `getStarted()` function, the first step is to login and get a `User` object.
2. Next, we open up a synced Realm and we add `partial: true` to the sync configuration. This is how you specify a partially synced Realm!
3. After we open the partially synced Realm, initially there is no data. The next step is to register a query. To do so, requires using a new API `Realm.subscribeToObjects()` We check if there are already objects in the Realm and if not, we subscribe with the query: `price > ${greaterThanPrice}`.
4. Finally, we register a listener on a query of the local Realm for all `Ticker` objects so that we can react to the new data that the server will partially sync. When new data arrives, we print the number of `Ticker`objects that were synced. After that, we will print out when any changes are made.

Now comment out `getStarted()` and instead run the partial sync function:

```javascript
// getStarted()

partialSyncTickers(10)
```

Given that we previously synced the entire `/tickers` Realm, we need to clear out our data locally by running:

```javascript
// Run in the same directory as `index.js`
rm -rf realm-object-server/
```

Now, simply re-run the `index.js` file and now you will instead just sync the data that matches the partial sync query!

```bash
node index.js
```

In Realm Studio, you will now see there is a partially synced Realm \(the number “10” will be a random ID\):  


![](https://s3.amazonaws.com/static.realm.io/docs/ros/images/getting-started-8.png)

Open the partially synced Realm:  


![](https://s3.amazonaws.com/static.realm.io/docs/ros/images/getting-started-9.png)

You will see that there is only the number of `Ticker` objects in the Realm that match the query! Try editing the `companyName` again for one of the `Ticker` objects which will print to your console:  


![](https://s3.amazonaws.com/static.realm.io/docs/ros/images/getting-started-10.png)

Congrats!

You have partially synced your first Realm. The same `subscribeToObjects()` API is provided in all Realm SDKs, so we recommend as a next step to try building a mobile application with partial sync _\(sample apps will be provided in the future\)_.

## What’s next?

If you’ve never used the Realm Database before, you might want to [start by picking your platform and checking the docs](https://realm.io/docs/). There, you’ll learn about what makes our database unique, like the power of live objects, auto-updating queries, and the freedom from ever having to use an ORM again. Or, you can follow a [tutorial to build an iOS app similar to RealmTasks from scratch](create-my-first-app.md).





Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

