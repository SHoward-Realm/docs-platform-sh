# Deleting Realms

As you're developing with the Realm Platform, there may be a time when you want to completely remove a Realm database from your Realm Object Server. This is very easy to do. All it takes is a simple REST call.

## Warning about Deleting Realms

{% hint style="danger" %}
You should never delete a realm while your app is in Production. Deleting a realm is a great way to clean up data while you learn and develop from realm. Before deleting the Realm from ROS, make sure that any / all clients \(iOS, Android, Node etc.\) has already deleted the app or database locally. If this is not done, they will try to upload their copy of the database - which might have been replaced in the meantime. 
{% endhint %}



## Deleting a Realm with Server Side Code

If you've created a server side app with `ros init` you can edit the `server.ts` file with the following code. Realm Object Server comes with a dependency called `superagent` that allows you to make easy http requests.

This example code will attempt to delete the realm `/abc/products`

```javascript
import RealmObjectServer = from 'realm-object-server'
import * as path from 'path'
import * as superagent from 'superagent'

const server = new RealmObjectServer.BasicServer()

const realmPath = `/8ec309f8cdbc42e8583d427f0ef91f81/products`

server.start({
        // This is the location where ROS will store its runtime data
        dataPath: path.join(__dirname, '../data')
    })
    .then(() => {
      const urlEncodedPath = encodeURI(realmPath)
      return superagent.delete(`http:${server.address}/realm/files/${urlEncodedPath}`)
        .set({ Authorization: server.adminToken })
    })
    .then((response) => {
      console.log(`Finished deleted ${response.body}`)
    })
    .catch(err => {
        console.error(`Error with Realm Object Server: ${err.message}`)
    })
```

## Deleting a Realm with a HTTP DELETE call

You can delete a Realm in the Realm Object Server by making an HTTP DELETE in the form of `http[s]://IP:PORT/realms/files/:realmPath` with `realmPath`being the path to the realm such as `/<userId>/myRealm` which is URI encoded.

This is the following example 

**Let's say I had a database at the path \(in a locally running ROS instance\): **

```text
/abc/products
```

**An admin token of**

```text
eyJhcHBfaWQiOiJpby5yZWFsbS5hdXRoIiwiaWRlbnRpdHkiOiJfX2FkbWluIiwiYWNjZXNzIjpbImRvd25sb2FkIiwidXBsb2FkIiwibWFuYWdlIl0sInNhbHQiOiI1MjYyNTUyMyJ9:ii1TmI40IE4pI2NO7xR/ZDLjq9ufKoRSKf+9yH3RALe+r9hpJxha2f8WBe8TmJgvNwBSlaIfCaH1nK19xirrv8BM9DLfowI25AaK+IXj0ylXD4tqsCpo4UkAagbgWCc/zKpj9YGzQJV22JYKtv5Q7Tdg9GVWev5dWQi/m+JY7tArYaaH46sxOYFsEHKxJ+Oi/KXXZu4KhzcssWf5n4bbgHDICl/CA9gX97r3DxPrn0AVlqKDcVZbAqlEIQhzzgrsoMlhUa9KaTxiXVvAIkS8X0axDtHMAwndNhcKnfJIK7wOg43tbsqcoVQQUoO4X/voDb+FZWnlNNl5mvRnvSDKhw==
```

**I could delete that database using that token by making an HTTP DELETE call to**

```http
http://127.0.0.1:9080/realms/files/abc%2Fproducts
## Note the encoded slash in the database path $userId%2Fdatabase
```

**Full example using wget**

```bash
$ wget --method DELETE --header 'Authorization: MY_ADMIN_TOKEN' http://127.0.0.1:9080/realms/files/abc%2Fproducts
```

**Output**

```bash
--2017-11-08 09:17:29-- http://127.0.0.1:9080/realms/files/8ec309f8cdbc42e8583d427f0ef91f81%2Feventblank

Connecting to 127.0.0.1:9080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2 [application/json]
Saving to: ‘abc%2Fproducts’
```



Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

