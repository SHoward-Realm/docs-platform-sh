# Serving Static Files

**Serving Static Files**

You can serve static files from a service by using a `@BaseRoute` with a `@ServeStatic` decorator. For example, to serve a directory or path, you could use the following snippet:

```text
import { BaseRoute, ServeStatic } from 'realm-object-server'
import * as path from 'path'
@BaseRoute('some')
@ServeStatic(path.join(__dirname, 'assets/index.html'))
class SomeClass {

}
```

You can now reach the `index.html` file at: `http://localhost:9080/some`



Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

