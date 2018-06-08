# Writing a Service in TypeScript

  
We highly recommend when writing a custom service in TypeScript ensure that your tsconfig.json's compiler options has

```text
"emitDecoratorMetadata": true,
"experimentalDecorators": true
```

It should look relatively similar to:

```text
{
  "compilerOptions": {
    "target": "es6",
    "module": "commonjs",
    "moduleResolution": "node",
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true
    ... other options
  }
}
```

If you are using `ros init`. You should have no problem with the generated `tsconfig.json` file in your project.

#### Desiging your Service with GET, PUT, POST, DELETE

Remember, a service is just an object. You can easily write a `ProductsService` like so:

```text
class ProductsService { 

}
```

You can then add the service to the server:

```text
server.addService(new ProductsService())
```

However this service doesn't really do anything! Let's give it some `REST` behavior! First import some the HTTP decorators from the `realm-object-server` package:

```text
import { 
  Get, 
  Post, 
  Put, 
  Delete, 
  BaseRoute
} from 'realm-object-server'
```

Now you can decorate your `ProductsService` class with an expressive set of functions. **In order for the service to have HTTP capabilities the service must have a @BaseRoute decorator**. In this example we will have a `ProductsService` that returns JSON objects with an HTTP GET request

```text
@BaseRoute('/products')
class ProductsService {

  products = [{
      productId: 1,
      name: 'toyota',
      price: 26000.25
    }, {
      productId: 2,
      name: 'book',
      price: 20.00
    }]

  @Get('/')
  getAllProducts(){
    return this.products
  }
}
```

You can now make a GET request to return your the products JSON at `http://localhost:9080/products`. Services can return normal objects \(which will be serialized into JSON\) or a `Promise<any>`, which will be serialized after the promise is resolved.

Thus the `getAllProducts()` method _could_ look like this:

```text
  @Get('/')
  getAllProducts(){
    return new Promise((resolve, reject => {
      setTimeout(() => {
        return this.products
      }, 2000)
    })
  }
```

You can even add URL parameters \(using `@Params('param')` and query string \(`@Query('querykey')`\) decorators to methods. Parameters can be defined with a `/:someparam`. Multiple params can be chained with `/:someparamone/:someparamtwo`

```text
  @Get('/:productId')
  getProductById(@Params('productId') productId: string){
    return this.products.find((p) => p.productId === productId))
  }
```

An example with multiple params:

```text
  @Get('/:productId/:name')
  getProductById(@Params('productId') productId: string, @Params('name') name: string){
    return this.products.find((p) => p.productId === product && c.name === name))
  }
```

An example with multiple params:

```text
  @Get('/:productId')
  getProductById(@Params('productId') productId: string, @Query('limit') limit: string){
    let limitNumber: number = parseInt(limit)
    return this.product.filter((c) => c.productId === productId && c.name === name))
      .limit(limitNumber)
  }
```

All returned values will have an HTTP status code of `200` and will be serialized with a JSON body. If you need fine grained control over the message that you are returning you can even choose to pass in the raw express request and response with `@Request` and `@Response`

```text
  @Get('/:productId')
  getProductById(@Request req, @Response res){
    res.status(203).send({
      productId: 'something',
      name: 'somethingelse'
    })
  }
```

```text
import { 
  Get, 
  Post, 
  Put, 
  Delete, 
  BaseRoute, 
  Body, 
  Request, 
  Response 
} from 'realm-object-server'

@BaseRoute('/product')
class ProductsService {

  products = [{
      productId: 1,
      name: 'toyota',
    }, {
      productId: 2,
      name: 'ford',
    }]

  @Get('/all')
  getAllProducts(){
    return this.products
  }

  @Post('/')
  async addNewProduct(@Body body: any) {
    this.products.push(body.product)
    return body
  }

  @Put('/:productId')
  async updateProduct(@Body body: any) {
    this.products.push(body.product)
    return body
  }

  @Delete('/:productId')
  async deleteProduct(@Param productId: string) {
    for(let product of this.products) {
      if (product.productId === productId) {
        this.products.splice(this.products.indexOf(product), 1)
      }
    }
  }
}
```

#### Writing a HTTP Service that uses Realm

If you want a service that uses a `Realm` instance you can easily get realms with a few modifications to the code above.

1. First import `ServerStarted` and `Realm` into your service file

```text
import { 
  ServerStarted,
  // ... other imports
} from 'realm-object-server'
import * as Realm from 'realm'
```

1. Create or Reference your object Schema and get a reference to the Realm.

The `@ServerStarted` decorator is fired when the server has started. This is a place where you can asynchronously open a Realm file of your choice via a `remotePath` and a remote url

So let's say we have a schema like so:

```text
const ProductSchema: Realm.ObjectSchema = { 
  name: 'Product',
  primaryKey: 'productId',
  properties: { 
    productId: 'int',
    name: 'string',
    price: 'float'
  }
}
```

We can get a `Realm` asynchronously with

```text
@BaseRoute('/products')
class ProductsService { 

  realm: Realm

  @ServerStarted()
  async serverStarted(server: Server) {
    this.realm = await this.server.realmFactory.open('/products', [ProductSchema])
  }
} 
```

1. Write a service to interface HTTP with your Realm Object

Let's say we want to get a Product by it's primary key and get all objects via a certain query

```text
import {
  BaseRoute,
  Get,
  ServerStarted,
  Server,
  Query
} from 'realm-object-server'

import * as Realm from 'realm'


@BaseRoute('/products')
class ProductsService { 

  realm: Realm

  @ServerStarted()
  async serverStarted(server: Server) {
    this.realm = await this.server.realmFactory.open('/products', [ProductSchema])
  }
  
  @Get('/')
  async get(Query('filter') filterQuery: string) {
    return this.realm.objects("Product").filtered(filterQuery).slice()
  }
  
  @Get('/:productId')
  async getByProductId(productId: string) {
    return this.realm.objectByPrimaryKey("Product", productId)
  }
} 
```

> Notice the slice. Slice turns Realm Objects into detached objects which makes it easily JSON serializable.



Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

