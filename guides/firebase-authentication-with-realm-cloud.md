# Firebase Authentication with Realm Cloud

Firebase provides a safe and secure authentication system. Firebase even have ready built UI components you can use which will save you many developer hours. This guide will show you how to connect your existing Firebase-powered Authentication system to use with Realm Cloud.   


## Prerequisites: 

This guide assumes you are already using Firebase Authentication in your app. A detailed guide on how to set it up can be found [in the Firebase documentation](https://firebase.google.com/docs/auth/). It will use Firebase Cloud Functions and this assumes that you are familiar with node/javascript

## Configuring the Backend

### Generate a public/private key pair

This can easily be done by following [this SSH guide](https://www.ssh.com/ssh/keygen/).  These will be needed to generate secure JWTs. 

### Enable JWT Authentication on your Realm Cloud Instance

Head over to your [cloud dashboard](https://cloud.realm.io/).  After logging in, select the instance for your project.  Select `Settings` and then enable JWT authentication.  You will need to paste in the public key which you generated above in the pem format.  

### Create a Firebase Cloud Function to generate a JWT for the user

Firebase cloud functions allows your mobile apps to call functions in the cloud without even having a server. To get started with firebase cloud functions [a detailed guide can be found here](https://firebase.google.com/docs/functions/get-started). If you aren't already familiar, you may want to follow their hello world example to get started. Once you have familiarized yourself with Firebase Cloud Functions, you can add a new function that will generate JWT for the signed in user. This function can then be called by using the firebase client libraries for Android/iOS/Javascript. A firebase cloud function for generating a JWT that can be used to sign in to realm cloud cloud look like this:

```javascript
const functions = require("firebase-functions");
const jwt = require('jsonwebtoken');
const fs = require('fs');
const key = fs.readFileSync(’pathToMyPrivateKeyFile');
exports.myAuthFunction = functions.https.onCall((data, context) => {    
    const uid = context.auth.uid, 
    const payload = { userId: uid },    
    const token = jwt.sign(payload, { key: key, passphrase: "your-passphrase" }, { algorithm: 'RS256'}),    
    return { token:token }
});
```

Within your `package.json`, you will need to add firebase-admin, firebase-functions, fs, and jswonwebtoken as dependencies. 

### Deploy the cloud function 

This is all you need on the server side. Now it’s time to connect your client to Realm Cloud!  


## Connecting your client to Realm Cloud

{% hint style="info" %}
This guide assumes you are using Android, but the process will be similar on iOS/Javascript. 
{% endhint %}

### Add Firebase Auth to your Project 

The easiest way to get started adding the Firebase sign in is by [following their guide here](https://firebase.google.com/docs/auth/android/firebaseui).  

### Add Firebase Cloud Functions to your project 

Next you simply need to add cloud functions to your project so that you can sign in.  Here is their [getting started guide](https://firebase.google.com/docs/functions/get-started).  

### Connect to Realm Cloud to sign in a user 

When signed in you need to call your firebase cloud function that generates a jwt to be used with Realm Cloud. This can be done using the cloud functions client libraries. 

The following example assumes Kotlin for Android, but it can be easily modified for other mobile clients: 

```kotlin
FirebaseFunctions.getInstance().getHttpsCallable(”myAuthFunction").call(args).addOnCompleteListener{    
    if(it.isSuccessful){        
        val result = it.result.data as Map<String,String>        
        val realmToken = result.get("token")        
        val syncCredentials = SyncCredentials.jwt(realmToken)
        doAsync (exceptionHandler = {            
            //Handle error        
        }) {            
            val user = SyncUser.logIn(syncCredentials, AUTH_URL)            
            //Done! You now have a sync user on Realm Cloud!        }
    } else {        
        //Failed to get sign in token    
    }}
})

```

