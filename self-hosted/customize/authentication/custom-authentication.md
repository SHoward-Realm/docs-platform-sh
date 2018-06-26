# Custom Authentication

Chances are you’d like to integrate your own authentication mechanism to identify your users. Realm Object Server supports authenticating users using a custom auth subclass. Let’s imagine that you have a custom REST endpoint to authenticate your company’s custom logic and it looks like this:

```text
POST http://mycompany.com/auth
{
  "departmentId": "1234abc",
  "email" "joe@mycompany.com"
  "pin": 1234
}
```

And say it returns to you a 200 success response like:

```text
HTTP Response 200
{
  "companyUserId": "987abc",
  "profileInfo":  {
    "firstName": "Joe",
    "lastName": "Budden",
    "level": "Director"
  }
}
```

In NodeJS we can use [superagent](https://github.com/visionmedia/superagent) a popular library to make HTTP JSON Requests to this endpoint.

```typescript
import { post } from 'superagent'

post('http://mycompany.com/auth')
    .send({
      "departmentId": "1234abc",
      "email": "joe@mycompany.com",
      "pin": body.pin
    })
    .then(successJSON => {

    })
    .catch(err => {

    })
```

With ROS we can wrap this entire flow into a custom AuthProvider

1. In your `index.ts` file import and extend the `AuthProvider` class. You will also need the `errors` namespace 

```typescript
import { BasicServer, auth, User, errors } from 'realm-object-server'

class MyCustomAuthProvider extends auth.AuthProvider {

  name = "mycustomauthprovider"

  authenticateOrCreateUser(body: any): Promise<User> {
    // body is the incoming JSON body from a Realm client request
  }
}
```

    2. Wrap the `superagent` post JSON call within the `authenticateOrCreateUser` . Use this to post to your end point. 

```typescript
import { BasicServer, AuthProvider, User, errors } from 'realm-object-server'
import { post } from 'superagent'

class MyCustomAuthProvider extends auth.AuthProvider {

  name = "mycustomauthprovider"

  authenticateOrCreateUser(body: any): Promise<User> {
    const departmentId: string = body.departmentId;
    const email: string = body.email
    const pin: string = body.pin
    return post('http://mycompany.com/auth')
      .send({
        "departmentId": "1234abc",
        "email": body.email,
        "pin": body.pin
      }).then(successResponse => {
        return this.service.createOrUpdateUser(
          successResponse.body.companyUserId,
          this.name, // this is the name of the provider,
          false, // this is if the user should or should not be an admin
          successResponse.body.profileInfo
        )
      })
  }
}
```

    3. Make sure to handle errors

In the event that the custom authentication fails, maybe due to bad credentials or some other reason, then make sure to reject the invalid credentials error.

In the catch block, return an error with a detail like so:

```typescript
import { BasicServer, auth, User, errors } from 'realm-object-server'
import { post } from 'superagent'

class MyCustomAuthProvider extends auth.AuthProvider {

  name = "mycustomauthprovider"

  authenticateOrCreateUser(body: any): Promise<any> {
    // body is the incoming JSON body from a Realm client request

    // Here we make a very simple call 
    return post('http://mycompany.com/auth')
      .send({
        "departmentId": "1234abc",
        "email": body.email,
        "pin": body.pin
      })
      .then((successResponseJSON) => { 
        return this.service.createOrUpdateUser(
          successResponseJSON.body.companyUserId
          this.name, // this is the name of the provider,
          false, // this is if the user should or should not be an admin
          successResponseJSON.profileInfo
        )
      })
      .catch(err => {
        return errors.realm.InvalidCredentials({
            detail: `Oh! No your information was wrong`
        })
      })
  }
}
```

    5. Add the custom auth provider to the server instance

```typescript
server.start({
  // This is the location where ROS will store its runtime data
  authProviders: [ new MyCustomAuthProvider() ]
})
```

    6. Try out your authentication with a client login

{% tabs %}
{% tab title="Javascript" %}
In Realm-JS you can call your custom authentication handler with the `Realm.Sync.User.registerWithProvider`

```javascript
import * as Realm from 'realm'

return Realm
  .Sync
  .User
  .registerWithProvider(`http://localhost:9080`, 
    { provider: 'mycustomauthprovider', providerToken: null, userInfo: {
      "email": "joe@mycompany.com",
      "pin", 1234
    }})
```
{% endtab %}

{% tab title="Realm Studio" %}
While attempting to login via Realm Studio, select the "Other" tab and login with a request body like: 

```javascript
{
    "provider": "mycustomauthprovider",
    "userInfo": {
        "email": "joe@mycompany.com",
        "pin": 1234
        }
}
```
{% endtab %}
{% endtabs %}

## Specifying a custom userId {#specifying-a-custom-userid}

In the event you want to specify your own custom userId instead of the automatically generated userId, then you can add the final optional parameter to set the userId to your liking. Note only userIds with characters of alpha-numeric \(0-9, A-Z, a-z\), “\_”, and “-“ are allowed.

```javascript
this.service.createOrUpdateUser(
  successResponseJSON.companyUserId
  this.name, // this is the name of the provider,
  false // this is if the user should or should not be an admin
  successResponseJSON.profileInfo,
+  "MYCUSTOMUSERID123"
)
```

Upon login in your client SDKs, your `Realm.Sync.User` or `SyncUser` identity will be `MYCUSTOMUSERID123`.



Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

