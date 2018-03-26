# MSSQL Data Connector

This guide walks you through how to use Realm’s data adapter to sync data from the Realm Object Server to a Microsoft SQL Server and vice versa. This guide shows the installation of the data adapter on an Ubuntu Server and assumes communication with an MSSQL Server in Amazon RDS, but any MSSQL Server will work as long as you are able to enable change tracking and snapshot isolation.  

## Requirements {#postgres}

* A Running Instance of the Realm Object Server
* A Realm [Feature Token](https://realm.io/trial/realm-professional-edition/)
* MSSQL Server 2014 or newer 
* The ability to enable Change Tracking and Snapshot Isolation on your MSSQL Server
* A SQL User with admin privileges \(to run the adapter\)

## Microsoft SQL Server {#postgres}

If you are running through this tutorial, we assume that you have a running MSSQL Server.  If you do not, it is simple enough to create one using [Amazon RDS](https://aws.amazon.com/rds/sqlserver/).  

### Loading in Test Data: 

To get you up and running, we will provide some test data to show how the data synchronization works.  You can run these SQL statements from any SQL client like SSMS or SQLPro for MSSQL.

#### Create the Database

```text
IF (SELECT db_id('SQLSyncDemo')) IS NULL CREATE DATABASE SQLSyncDemo;
```

#### Enable Change Tracking on the Database

```text
ALTER DATABASE SQLSyncDemo  
SET CHANGE_TRACKING = ON  
(CHANGE_RETENTION = 2 DAYS, AUTO_CLEANUP = ON)  

ALTER DATABASE SQLSyncDemo
SET ALLOW_SNAPSHOT_ISOLATION ON
GO

USE SQLSyncDemo  
GO
```

#### Create Tables

```text
CREATE TABLE dbo.Address (
	ID int NOT NULL,
	ZipCode varchar(10) NOT NULL
);

ALTER TABLE dbo.Address ADD CONSTRAINT PK_Address PRIMARY KEY (ID);

CREATE TABLE dbo.Customer (
	ID int NOT NULL,
	Company varchar(100) NULL
);

ALTER TABLE dbo.Customer ADD CONSTRAINT PK_Customer PRIMARY KEY (ID);

/* JOIN table */
CREATE TABLE dbo.CustomerAddress (
	CustomerID int NOT NULL,
	AddressID int NOT NULL,
	IsBilling bit NOT NULL
);

ALTER TABLE dbo.CustomerAddress ADD CONSTRAINT PK_Customer_Address PRIMARY KEY (CustomerID, AddressID); /* Compound PK */

CREATE TABLE dbo.Estimate (
	ID int NOT NULL,
	customerid int NOT NULL,
	something varchar(50) NULL
);

ALTER TABLE dbo.Estimate ADD CONSTRAINT PK__Estimate PRIMARY KEY (ID);

/* INSERT DATA */

INSERT INTO Address (ID, ZipCode) VALUES 
(1,'90101'),
(2,'80100'),
(3,'90100'),
(4,'80100'),
(6,'91111');

INSERT INTO Customer (ID, Company) VALUES 
(1,'C2'),
(2,'C2'),
(3,'C3');

INSERT INTO CustomerAddress (CustomerID, AddressID, IsBilling) VALUES 
(1,1,0),
(1,2,0),
(1,4,0),
(2,3,0),
(3,6,1);

INSERT INTO Estimate (ID, customerid, something) VALUES 
(13,1,'Some estimate');
```

#### Enable Change Tracking on each Table

```text
ALTER TABLE Address ENABLE CHANGE_TRACKING WITH (TRACK_COLUMNS_UPDATED = ON);
ALTER TABLE Customer ENABLE CHANGE_TRACKING WITH (TRACK_COLUMNS_UPDATED = ON);
ALTER TABLE CustomerAddress ENABLE CHANGE_TRACKING WITH (TRACK_COLUMNS_UPDATED = ON);
ALTER TABLE Estimate ENABLE CHANGE_TRACKING WITH (TRACK_COLUMNS_UPDATED = ON);
```

## The Data Adapter

The data adapter relies on a number of components for two way data synchronization.  

* Data Adapter Package: This contains the core codebase of the adapter
* Adapter.js: This runs as a process to facilitate synchronization 
* realmmodels.js: This defines the schema to be synchronized 
* config.js: this defines a number of variables which are used during adapter configuration 
* loader.js: You may choose to perform a one time bulk import of data into the Realm Object Server before running the adapter 

### Setup the Data Adapter Package

{% hint style="info" %}
You'll need to contact [sales@realm.io](mailto:sales@realm.io) to receive the MSSQL Data Adapter Package file.
{% endhint %}

Find a desirable location on your server and run the following commands from your terminal: 

```bash
mkdir mssql-test
cd mssql-test
#press enter through all the prompts that are returned to create a project
npm init
npm install realm@2.1.0
#replace the path below with your local path to your adapter package
npm install ~/Downloads/realm-mssql-adapters-1.0.7-1.tgz
```

### Prepare your Config File 

Create the file by running the following command: 

```text
touch config.js
```

Using your preferred text editor, paste the following into your config file: 

{% code-tabs %}
{% code-tabs-item title="config.js" %}
```javascript
module.exports = {
    // Database name
    database_name: 'SQLSyncDemo',

    database_schema: 'dbo',

    // Realm Object Server URL
    realm_object_server_url: 'realm://localhost:9080',

    auth_server_url: 'http://localhost:9080',

    // Admin token used to talk to ROS. Get it from
    // `data/keys/admin.json` under the root dir of ROS.
    admin_user_token: 'eyJhcHBfa....',
    
    //Get your feature token from a member of the Realm team 
    feature_token: 'eyjhcd.....', 

    admin_username: 'realm-admin',
    admin_password: '',

    // The synced Realm path for the data
    target_realm_path: '/SQLDemoNew',

    // Postgres config used for all connections - replace with your data
    sqlserver_config: {
        user:       'realm',
        password:   'my-sql-password',
        server:     'mydbinstanceAddress.us-east-1.rds.amazonaws.com',
        port:        1433
    },
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

You'll need to change a number of the various variables to match your own environment like the SQL Server information, admin user token, and the ROS address.  

### Prepare your Models File

Create the file by running the following command: 

```text
touch realmmodels.js
```

Using your preferred text editor, paste the following into your models file:

{% code-tabs %}
{% code-tabs-item title="realmmodels.js" %}
```javascript
const Customer = {
    name: 'Customer',
    primaryKey: 'ID',
    properties: {
        ID: 'int',
        Company: 'string',
        addresses: { type: 'list', objectType: 'CustomerAddress', watch_table: true },
        Estimates: { type: 'list', objectType: 'Estimate', watch_table: true }, 
    }
}

const CustomerAddress = {
    name: 'CustomerAddress',
    primaryKey: 'pk',
    properties: {
        pk: 'string',
        customerid: { type: 'linkingObjects', objectType: 'Customer', property: 'addresses' },
        addressid: 'Address',
        IsBilling: 'bool',
    }
}

const Address = {
    name: 'Address',
    primaryKey: 'ID',
    properties: {
        ID: 'int',
        ZipCode: 'string',
    },
}

const Estimate = {
    name: 'Estimate',
    primaryKey: 'ID',
    properties: {
        ID: 'int',
        customerid: { type: 'linkingObjects', objectType: 'Customer', property: 'Estimates' },
        Something: 'string',
    }
}


module.exports = [
    Customer, 
    CustomerAddress, 
    Address, 
    Estimate
];
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
This models file is built specifically to work with the SQL data which we loaded into our SQL Server.  You will edit this schema to match your own data that you wish to synchronize.  
{% endhint %}

### Perform a one time import of the existing SQL Data into ROS  

Create your loader file by running the following command: 

```text
touch loader.js
```

Using your preferred text editor, paste the following into your loader file:

{% code-tabs %}
{% code-tabs-item title="loader.js" %}
```javascript
const Realm = require('realm');
const fs = require('fs');
const path = require('path');
const SQLServerRealmLoader = require('realm-mssql-adapters').SQLServerRealmLoader;
const Config = require('./config');

const Models = require('./realmmodels');

if (process.env.NODE_ENV !== 'production') {
    require('source-map-support').install();
}

// Print out uncaught exceptions
process.on('uncaughtException', (err) => console.log(err));

// Unlock Professional Edition APIs
Realm.Sync.setFeatureToken(Config.feature_token);
let admin_user = Realm.Sync.User.adminUser(process.env.ROS_ADMIN_TOKEN || Config.admin_user_token, Config.auth_server_url);

const conf = {
    // Realm configuration parameters for connecting to ROS
    realmConfig: {
        server: Config.realm_object_server_url, // or specify your realm-object-server location
        user:   admin_user,
    },
    dbName: Config.database_name,
    dbSchema: Config.database_schema,
    // SQL Server configuration and database name
    sqlserverConfig: Config.sqlserver_config,
    resetPostgresReplicationSlot: true,


    // Set to true to create the SQL Server DB if not already created
    createSQLServerDB: false,
    initializeRealmFromSQLServer: false,

    // Set to true to indicate SQL Server tables should be created and
    // properties added to these tables based on schema additions
    // made in Realm. If set to false any desired changes to the
    // SQL Server schema will need to be made external to the adapter.
    applyRealmSchemaChangesToSQLServer: false,

    // Only match a single Realm called 'testRealm'
    //realmRegex: `/*`+Config.database_name,
    realmRegex: Config.target_realm_path,

    // Specify the Realm name all SQL Server changes should be applied to
    mapSQLServerChangeToRealmPath: Config.target_realm_path,

    // Specify the Realm objects we want to replicate in SQL Server.
    // Any types or properties not specified here will not be replicated
    schema: Models,

    printCommandsToConsole: true,

    compoundPrimaryKeys: {
        'CustomerAddress': [
            { property: 'customerid', dbColumn: 'CustomerID', type: 'int' },
            { property: 'addressid', dbColumn: 'AddressID', type: 'int' },
        ]
    }
}

var loader = new SQLServerRealmLoader(conf);
loader.init().catch(console.error)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Now you should be able to run the loader to import your SQL data into the Realm Object Server.  Simply run:

```text
node loader.js 
```

{% hint style="info" %}
For improved performance with JOIN tables, seed the RealmId field with a uuid on the SQL side before running the loader script.
{% endhint %}

You should now be able to use Realm Studio to see that the data was synchronized to your Realm Object Server.  

![MSSQL Data Loaded into ROS](../../../.gitbook/assets/screen-shot-2018-02-16-at-2.18.51-pm.png)

### Run the Adapter for Bidirectional Sync

Finally, we need to create and configure the adapter file which will facilitate bidirectional synchronization.  Create the file by running the following command: 

```text
touch adapter.js 
```

Using your preferred text editor, paste the following into your adapter file:

{% code-tabs %}
{% code-tabs-item title="adapter.js" %}
```javascript
const fs = require('fs');
const path = require('path');
const process = require('process');
const SQLServerAdapter = require('realm-mssql-adapters').SQLServerAdapter;
const Config = require('./config');
const Models = require('./realmmodels');

// Print out uncaught exceptions
process.on('uncaughtException', (err) => console.log(err));

// Unlock Professional Edition APIs
Realm.Sync.setFeatureToken(Config.feature_token);
const admin_user = Realm.Sync.User.adminUser(process.env.ROS_ADMIN_TOKEN || Config.admin_user_token, Config.auth_server_url);

const conf = {
    // Realm configuration parameters for connecting to ROS
    realmConfig: {
        server: Config.realm_object_server_url, // or specify your realm-object-server location
        user:   admin_user,
    },
    dbName: Config.database_name,
    dbSchema: Config.database_schema,
    // SQL Server configuration and database name
    sqlserverConfig: Config.sqlserver_config,

    // Set to true to create the SQL Server DB if not already created
    createSQLServerDB: false,
    initializeRealmFromSQLServer: false,

    // Set to true to indicate SQL Server tables should be created and
    // properties added to these tables based on schema additions
    // made in Realm. If set to false any desired changes to the
    // SQL Server schema will need to be made external to the adapter.
    applyRealmSchemaChangesToSQLServer: false,

    // Only match a single Realm called 'testRealm'
    //realmRegex: `/*`+Config.database_name,
    realmRegex: Config.target_realm_path,

    // Specify the Realm name all SQL Server changes should be applied to
    mapSQLServerChangeToRealmPath: Config.target_realm_path,

    // Specify the Realm objects we want to replicate in SQL Server.
    // Any types or properties not specified here will not be replicated
    schema: Models,

    printCommandsToConsole: false,

    compoundPrimaryKeys: {
        'CustomerAddress': [
            { property: 'customerid', dbColumn: 'CustomerID', type: 'int' },
            { property: 'addressid', dbColumn: 'AddressID', type: 'int' },
        ]
    }
}

var adapter = new SQLServerAdapter(conf);
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Finally, run the adapter file: 

```text
node adapter.js
```

{% hint style="info" %}
In a production environment, make sure to run the adapter in the background using a process manager like PM2.  
{% endhint %}

You can now make changes on either the MSSQL or ROS side and see the changes synchronized between the two databases.  

