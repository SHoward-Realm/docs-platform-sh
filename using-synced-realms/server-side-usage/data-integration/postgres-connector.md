# Postgres Connector

This guide walks you through how to use Realm’s data adapter to sync data from the Realm Object Server to a Postgres Server and vice versa. This shows an installation on a bare CentOS server of both Postgres and the Realm Data Adapter but any other platform that allows the enabling of logical replication could also be used. If you already have an existing Postgres server you can skip to the step that starts with installing the Realm Data Adapter.

## Prerequisites: {#prerequisites:}

* Postgres 9.x \(we recommend using 9.6\)
* Realm Object Server 2.x \(or higher\) or Realm Cloud

## Postgres {#postgres:}

{% tabs %}
{% tab title="Manual Install via CentOS" %}
The first thing we need to do is install Postgres on a EC2 instance. SSH to a fresh server.

```bash
# First we need to set-up the latest Postgres repo

[centos@postgres-server ~]# rpm -Uvh [https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm](https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm)

# Now update the repo

[centos@postgres-server ~]# yum update

# Let’s install Postgres

[centos@postgres-server ~]# sudo yum install postgresql96-server postgresql96-contrib
```

### Now let’s initialize the Postgres server {#now-let's-initialize-the-postgres-server}

```bash
[centos@postgres-server ~]# /usr/pgsql-9.6/bin/postgresql96-setup initdb
```

Now we need to configure the Postgres server to accept the Realm data adapter connection. It is imperative to enable `logical_replication` as the adapter relies on this feature for data synchronization.

```bash
# Open the Postgres conf file and add the following lines

[centos@postgres-server ~]# vi /var/lib/pgsql/9.6/data/postgresql.conf

wal_level = logical

max_wal_senders = 8

wal_keep_segments = 4
max_replication_slots = 4
```

`Logical replication` is a built in streaming replication feature that allows clients to subscribe to the transaction logs. The transactions can then be parsed and used to generate Realm write transactions using the Realm Data Adapter. These are the settings we recommend for most installations but they could be changed based on your performance or architectural considerations.

You will also need to find the lines

```bash
#listen_addresses = 'localhost' 

#port = 5432
```

And uncomment them and configure the listening address and port. By using a \* on the address it will listen on every IP on the server which is useful in initial set-up for dev but you may want to lock this down in the future.

```bash
listen_addresses = '*' 

port = 5432
```

Now let’s edit the conf file to allow for password authentication and a remote replication host

```bash
[centos@postgres-server ~]# sudo vi /var/lib/pgsql/9.6/data/pg_hba.conf
```

Find the lines that look like this at the bottom and change them to use MD5 and add the IP address of your realm data adapter server

```bash
host    all     all        127.0.0.1/32        ident

host    all     all        ::1/128            ident

#host replication postgres 127.0.0.1/32 ident
```

Now they should look like this

```bash
host    all     all        0.0.0.0/0        md5

host    all     all        ::1/128            md5

host    replication     postgres        <IP_ADDRESS_OF_ADAPTER_SERVER>/32        md5
```

Now let’s enable and start our Postgres server

```bash
[centos@postgres-server ~]# systemctl enable postgresql-9.6

[centos@postgres-server ~]# systemctl start postgresql-9.6
```

Now let’s setup a password by switching to the postgres user

```bash
[centos@postgres-server ~]# su - postgres

#Enter the psql prompt

-bash-4.2$ psql

#Configure a password for the postgres user

postgres=# \password postgres
```
{% endtab %}

{% tab title="Quickstart via Amazon RDS" %}
[Amazon RDS](https://aws.amazon.com/rds/) provides a way to quickly spin up a database.

You'll want to start by creating a [Postgres instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.PostgreSQL.html). We suggest using Postgres 9.6. While creating the instance, spec it out however you like. Make sure to make note of the `DB instance identifier` , `master username` and `master password`. We will assume that we are using the default port of `5432`.

After creating the instance, you may [test connecting to it](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.CreatingConnecting.PostgreSQL.html#CHAP_GettingStarted.Connecting.PostgreSQL).

We will then need to enable `logical_replication` which facilitates the data synchronization with Realm.

To do this, you will need to create a new [parameter group](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.html) within your Amazon RDS. The parameter group family should match your version of Postgres \(i.e. postgres9.6\). You can keep all of the settings as the default except `rds.local_replication` which needs to be set to 1.

More information on logical replication in RDS can be found [here](https://aws.amazon.com/blogs/aws/amazon-rds-for-postgresql-new-minor-versions-logical-replication-dms-and-more/).
{% endtab %}
{% endtabs %}

You should now be able to connect to the Postgres server remotely. There are various tools you can use online to connect and browse Postgres such as [Postico](https://eggerapps.at/postico/) .

## Realm Data Adapter {#realm-data-adapter:}

{% hint style="info" %}
You'll need to contact [info@realm.io](mailto:info@realm.io) to receive the Postgres Data Adapter Package file.
{% endhint %}

Now let’s setup the Realm Data Adapter. We will use a remote CentOS server.  ****If you do not yet have a running instance of the Realm Object Server, sign up for a [cloud instance](https://cloud.realm.io/) or see instructions on how to [install a self-hosted Realm Object Server](../../../self-hosting/installation/). \(We've also included instructions if you've liked to run locally on a Mac\)

{% tabs %}
{% tab title="CentOS" %}
SSH to your new CentOS server for the data adapter

```bash
#Install EPEL which contains the node repos we need

[centos@data-adapter-server ~]# sudo yum -y install epel-release

#Install nodejs - it should come with npm

[centos@data-adapter-server ~]# sudo yum -y install nodejs

#We will also need to install node-gyp and postgresql-devel

[centos@data-adapter-server ~]# sudo yum -y install node-gyp

[centos@data-adapter-server ~]# sudo yum -y install postgresql-devel

#Now install Realm and the Realm Data Adapter npm package

[centos@data-adapter-server ~]# sudo npm install realm
#you will need to update your path and version information according to the package you received
[centos@data-adapter-server ~]# sudo npm install ~/Downloads/realm-postgres-adapters-1.0.10.tgz
```
{% endtab %}

{% tab title="macOS" %}
```text
#Now install Realm and the Realm Data Adapter npm package

#create a new npm project (name as you like)

npm init

npm install realm

npm install ~/Downloads/realm-postgres-adapters-1.0.10.tgz
```
{% endtab %}
{% endtabs %}

You can find the Adapter.js code [here](https://gist.github.com/mgeerling/9cca913539caef4d24f3d7ec6afe3daa).

And we use Config.js to set up your variables [here](https://gist.github.com/mgeerling/81ec8eba70accb0e2feae7155c49d82f).

When in development it may be necessary to reset your environment as you test things. You can use this [Reset.js script](https://gist.github.com/ianpward/6aa25df804a14b4a4d6e7156322a9747) to do so

Place them all in a new folder you create and let’s begin by editing `config.js` with your environment variables. The postgres variables should all come from your previous setup and you can find the admin token at `/data/keys` on the Realm Object Server. The last thing you will need to enter is `database_name:` If you are connecting to an already existing Postgres database then you want to match the name of your already existing database otherwise you can choose something unique and you can configure the Realm Data Adapter to automatically create that database and insert the schema and data being pulled from Realm Object Server.

Let’s take a look at the adapter.js code and look at some of the parameters we can set. Most of the code has to do with setting up the adapter for your environment - the first is:

```text
resetPostgresReplicationSlot?: boolean,
```

This optional config variable will reset the replication slot on the Postgres server when the adapter first boots up, this is useful for starting from the beginning.

The next two options you will see are:

```text
createPostgresDB?: boolean,

initializeRealmFromPostgres?: boolean,
```

The first option creates the Postgres database assuming it does not already exist using the configuration parameters you provided and creates tables using the realm object schema. The second option does the reverse, it creates the realms on the Realm Object Server matching the already existing Postgres schema. Typically, these are configured with one true and the other false. If you are adding a Postgres server to an already existing deployment of Realm Object Server you would set createPostgresDB to true and initializeRealmFromPostgres to false to ease in the setup of your Postgres server and minimize any schema mismatch issues. If you are adding Realm Object Server onto your existing Postgres database you will set createPostgresDB to false and initializeRealmFromPostgres to true for the same reasons.

The next config option you see is:

```text
customPostgresTypes?:
```

This config option accepts a dictionary of key value pairs in JSON format. Postgres supports the ability to create custom types, if you have these in your Postgres database that you will mapping to Realm then you need to create a corresponding type in Realm that corresponds to your custom Postgres type. You can find the supported Realm types [here](https://realm.io/docs/javascript/latest/#supported-types)

```text
applyRealmSchemaChangesToPostgres?: boolean
```

This option will execute SQL commands to extend the schema anytime the data model in the mapped Realm changes. It will create new tables or add new columns when new fields are added to your Realm objects.

```text
realmRegex:
```

This option takes a regular expression string and tells the adapter what Realms to monitor for changes.

Let’s say the realm you were trying to monitor is

```text
realm://127.0.0.1/myRealm
```

You could use this in the config:

```text
realmRegex: ‘^/myRealm$’
```

Or let’s say that each user is opening a realm with the URL realm:

```text
realm://127.0.0.1/~/myRealm
```

The `~` here expands into a realmId on the server to get the URL

`fb80255953a5eba491671781778d3e91/myRealm` for example. In this case there would be a realm created per user and you could match all of them with:

```text
realmRegex: ‘^/(.*?)/myRealm$’
```

The next config is:

```text
mapPostgresChangeToRealmPath:
```

Is used to map a table and its properties to the appropriate realm path. This can be either a string literal or a function that returns a string. If you were only replicating a single realm then you would pass in the same regular expression that you gave for realmRegex, for instance: ‘^/myRealm$’

However, in the case where each user opens their own instance of myRealm you will need to map the Postgres change to the correct realm user. You can do this by using a function like this:

```text
mapPostgresChangeToRealmPath: (tableName, props) => ‘/${props.userID}/myRealm’
```

You can also go the opposite way by using this config parameter:

```text
mapRealmChangeToPostgresTable?:
```

This option is less commonly used because a Realm object typically maps to a Postgres table - with each user’s realm entries mapping to rows in the appropriate table. However, in case you do need to map Realm changes to specific tables based on custom logic you can call a function like so:

```text
mapRealmChangeToPostgresTable?: (realmPath) => { tableName: string, extraProperties: any }
```

Given the realmPath that the change occurred on you must return the tableName in Postgres that you want to write the change to and a extraProperties Javascript object which allows mapping realm data into other Postgres columns. You want to support more columns to convert data that is in realm as a single property. Such as if you wanted to assign an ID in Realm to two columns in Postgres.

The last thing we need to configure is the schema for the objects and tables we are using the data adapter for. You can place the schema directly into `adapter.js` or more commonly you will export the schema from another Javascript file. See [this sample realmmodels.js file](https://gist.github.com/ianpward/727e93e8ec8e85218f0231ed1fb5ebfc).

There are a couple of things to notice when looking at the code and comparing it to the Postgres schema. The first thing you will notice is that not all the tables are reflected as Realm objects. By only listing a subset, the adapter will ignore the other tables and only sync Bus, Schedule, Driver, and Route tables. Additionally, any fields that you do not care about syncing can be dropped from the data model, you can see that fare is missing from Route schema. In order for the Postgres adapter to work all Object must have a primary key so that lookups can resolve in a timely manner.

The other important point demonstrated in this mapping is the Routeid property which in Postgres is a foreign key to the primary key in the Route table. However, instead of defining this as a string in Realm, we use Realm's link support to define this as a link to the corresponding Route object. The adapter will automatically resolve the foreign key relationship into a Realm link!

The data structure between Route and Schedule is that each Route row/object will link to 0 or more Schedule row/object\(s\). The number of linked Schedules represents the number of schedules of each route. Thus in the app as you add more schedules to a route object, the app will create Schedule object which the adapter will then convert into a new row in the Schedule table.

Similarly, the reverse will happen, such that if you remove a Schedule row from Postgres, the adapter will convert this change and delete the corresponding Schedule object in Realm.

Realm also has the ability to support List properties as shown [here](https://realm.io/docs/javascript/latest/#to-many-relationships).

When the data adapter encounters a List property it will go and create a subtable on Postgres with three columns: oid, idx, and target. The oid is the primary key of the main table, target is the primary key of the list table, idx is the index into an array.

The concept of lists does not typically exist in relational databases, the data adapter was designed to be one-way only for lists so if you want to maintain order you will need to use a stored procedure in Postgres. The indexes are zero-based and they need to be contiguous and unique. If you don't need ordered lists you can change your model to reference the parent object which means list properties are no longer needed

All of the default Postgres types are automatically mapped and converted to corresponding [Realm types](https://realm.io/docs/javascript/latest/#supported-types) as shown here:

| Realm type | Postgres types |
| --- | --- | --- | --- | --- | --- | --- |
| `string` | `text`, `varchar`, `character`, `character varying`, `tsvector`, `json`, `bytea`, `uuid` |
| `float` | `numeric`, `decimal`, `double precision` |
| `int` | `bigint`, `smallint`, `integer` |
| `bool` | `bool` |
| `date` | `timestamp`, `timestamp without time zone`, `timestamp with time zone`, `date` |
| `data` | not supported currently |

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3)

