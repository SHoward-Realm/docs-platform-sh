# Middlewares

Use of Middlewares on ROS 2.x is built ontop of express-js. 

You are able to put middlewares on:

1. the entire server
2. on one service
3. on one route

Middlewares are isolated to the level that you put them in. So if you only put `cors()` on one route of one service then you will only be hitting the `cors()` middleware.

For example, this applies the middleware to the entire server

```typescript
import * as cors from 'cors'
import * as path from 'path'

server.start({
    dataPath: path.join(__dirname, '../data')
    middlewares: [cors()]
});
```

Here’s an example of putting a middleware on just a single service. This will have no effect on `SyncProxy`, `RealmDirectory`, or `AuthService`. Any request to `http://localhost:9080/cars` will have cors\(\) middlewares being hit.

```typescript
import { Server, BaseRoute } from 'realm-object-server' 
import * as cors from 'cors'

@BaseRoute('/cars')
@MiddlewaresBefore(cors()) // this will hit the cors middleware only for CarsService
class CarService {
  //service code here
}

const server = new Server()
server.addService(new CarService())
server.start()
```

Here’s an example of putting a middleware on just a single route of a single service. Any request to `http://localhost:9080/cars/toyotas` will have cors\(\) middlewares being hit, while `http://localhost:9080/cars/hondas` will not.

```typescript
import { Server, BaseRoute, Get } from 'realm-object-server' 
import * as cors from 'cors'

@BaseRoute('/cars')
class CarService {
  //service code here

  @Get('/toyotas')
  @MiddlewaresBefore(cors())
  getToyotas() {
    // you'll only hit cors if you try 
  }

  @Get('/hondas')
  getHondas() {
    // no cors()
  }
}

const server = new Server()
server.addService(new CarService())
server.start()
```









Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

