# Create a ROS Instance on Bluemix Cloud Foundry

##  1. Install the Server 

{% hint style="info" %}
Node.js should be installed on the machine to run the following commands 

Trying running with `sudo` if you find that you don't have permission
{% endhint %}

```bash
#install the server
npm install -g realm-object-server

#create a new project 
ros init realm-object-server-app 
```

## 2. Update `src/index.ts` file from realm-object-server-app project with the port number 

Navigate to the `index.ts` file within your newly created realm-object-server-app project.  \(This is created in the directory where you executed `ros init`\) . 

```typescript
import { BasicServer } from 'realm-object-server'
import * as path from 'path'

const server = new BasicServer()

server.start({
        // This is the location where ROS will store its runtime data
        dataPath: path.join(__dirname, '../data')

        // The address on which to listen for connections
        // Default: 0.0.0.0
        // address?: string

        // The port on which to listen for connections
        // Default: 9080
        // port?: number = 9080
        port : process.env.PORT,
        
        //... 

    })
    .then(() => {
        console.log(`Your server is started `, server.address)
    })
    .catch(err => {
        console.error(`There was an error starting your file`)
    })
```

## 3. Create `manifest.yml` file to deploy realm-object-server-app on Bluemix Node.js runtime/build-pack 

The following excerpt is from a sample manifest.yml: 

```text
---
applications:
- name: samplenodejsapp
  random-route: true
  memory: 1024M
  host: samplenodejsapp
  domain: mydomain.us-west.mybluemix.net
```

## 4. Add dependency for node-pre-gyp package in `package.json` 

Edit your `package.json` to include the following: 

{% code-tabs %}
{% code-tabs-item title="package.json" %}
```javascript
"scripts": {
    "build": "rm -rf dist; ./node_modules/.bin/tsc",
    "clean": "rm -rf dist",
    "start": "npm run build && node dist/index.js"
},
"devDependencies": {
    "typescript":"2.5.3"
},
"dependencies: {
    "node-pre-gyp":"*",
    "realm-object-server":"^2.2.0"
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 5. Deploy application on Bluemix

Use the Eclipse tool or CF CLI tool: 

Sample URL: `http://samplenodejsapp.kpsj001.us-west.mybluemix.net/ `

## Additional Reference Links

* [Deploy a sample Node.js application on Bluemix](https://www.ibm.com/developerworks/cloud/library/cl-bluemix-fundamentals-create-and-deploy-a-node-app-to-the-cloud/index.html)
* [Sample manifest.yml ](https://github.com/cloudfoundry-samples/cf-sample-app-nodejs/blob/master/manifest.yml)



Not what you were looking for? [Leave Feedback](https://www.getfeedback.com/r/uO1Zl0vE)

