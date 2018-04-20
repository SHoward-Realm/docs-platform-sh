# Writing to a Server Realm

After a Realm has been opened, writing to it is as simple as: 

```javascript
const realm = server.openRealm('/products', [ProductSchema])
realm.write(() => {
    let product = realm.create('product', {
        productId: 1,
        name: 'widget',
        price: 9.99 
    });
})
```

If you're looking for some code to get started, the following javascript performs a basic username/password login to the Realm Object Server and then writes to a Realm.

```javascript
const BasicServer = require('realm-object-server').BasicServer

//Params to edit
var username = 'mycoolusername'; 
var password = 'password';

//set the Schema of your object to be stored in ROS -- depends on Kafka message
const ProductSchema = {
    name: 'Product',
    primaryKey: 'productId',
    properties: {
      productId: 'int',
      name: 'string',
      price: 'float'
    }
}
const server = new BasicServer()
server.start({
    // This is the location where ROS will store its runtime data
    dataPath: path.join(__dirname, '../data')
}).then(() =>{
    // server.address will have the host and port. We can use this to construct the login url.
    return await Realm.Sync.User.login(`http://${server.address}`, username, password)
})
.then(user => {
     return server.openRealm('/products', [ProductSchema])
})
.then(realm => {
    realm.write(() => {
        let newProduct = realm.create('Product', {
            productId: 1,
            name: 'widget',
            price: 9.99
        });
    })
})
.catch(error => {
    console.log('ROS Error' + error);
});
```



Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

