# IP Filtering Middleware

**IP Filtering Middleware**

There are many scenarios where you'd like to limit IPs. The common cases are:

1. I'd like to limit specific IP addresses from accessing Realm Object Server entirely
2. I'd like to limit specific IP addresses from accessing a service entirely

Since Realm Object Server allows you to use express middlewares, you are fully capable of using [express-ipfilter](https://www.npmjs.com/package/express-ipfilter) or [ip-regex](https://www.npmjs.com/package/ip-regex) on your entire server instance, an entire service instance, and/or just a service route.

The following examples will use `express-ip-filter`. You can read more about its custom settings [here](https://www.npmjs.com/package/express-ipfilter).

**Limiting the IP Address for your entire Server**

You apply a middleware to the entire server by setting the `middlewares` property on the `ServerConfig` you use with `server.start`. It can be set to either a single middleware or to an array of middlewares.

```text
import { IpFilter } from 'express-ipfilter'
import { Server } from 'realm-object-server'
import * as path from 'path'

const server = new Server();

const ips = ['127.0.0.1'];
server.start({
        dataPath: path.join(__dirname, '../data')
        middlewares: ipfilter(ips, {mode: 'allow'})
    })
    .then(() => {
        console.log('Only localhost (aka 127.0.0.1) can connect!');
    });
```

**Limiting the IP Address for just a service**

There are instances where you'd only like to limit IPs for a certain service. In the following snippet we have limited whitelisting of ips for the entire `CarsService`.

```text
import { IpFilter } from 'express-ipfilter'
import { MiddlewaresBefore, Server, BaseRoute, Get } from 'realm-object-server'

const ips = ['127.0.0.1'];
const middleware = ipfilter(ips, {mode: 'allow'});

@BaseRoute('/cars')
@MiddlewaresBefore(middleware)
class CarsService {

    @Get('/teslas')
    getTeslas(){
        // only whitelisted ips will enter in here
        return someJson;
    }
}

const server = new Server();
server.start({
        // Server configs
    })
    .then(() => {
        console.log('Only localhost (aka 127.0.0.1) can get cars');
    });
```

**Limiting the IP Address for just a service route**

If you just want to limit the IP address of a single service route but not the entire service, you can use the `MiddlewaresBefore` on a single route. In the following snippet we have limited whitelisting of ips only for the `getTeslas` method on `CarsService`.

```text
import { IpFilter } from 'express-ipfilter'
import { MiddlewaresBefore, Server, BaseRoute, Get } from 'realm-object-server'

const ips = ['127.0.0.1'];
const middleware = ipfilter(ips, {mode: 'allow'})

@BaseRoute('/cars')
class CarsService {
    @Get('/teslas')
    @MiddlewaresBefore(middleware)
    getTeslas() {
        // only whitelisted ips will enter in here
        return someJson;
    }
}

const server = new Server();
server.start(
        // Server configs
    )
    .then(() => {
        console.log('Only localhost (aka 127.0.0.1) can get teslas')
    });
```

Just like with any middleware, they will be processed in the following order:

1. `ServerConfig.middlewares`
2. `@MiddlewaresBefore` on a Service class
3. `@MiddlewaresBefore` on a Service class method



Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

