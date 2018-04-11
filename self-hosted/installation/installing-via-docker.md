# Installing via Docker

[Docker](https://docs.docker.com/install/) is a great tool for quickly getting a running instance.  We provide the following docker images and instructions for usage in a **development environment**

## Installing the Realm Object Server

If you haven't already, sign up for a Docker Hub account.  Once you've done this, simply run this statement from your command line: 

```bash
docker pull realm/realm-object-server:latest
```

## Running the Server 

The Realm Object Server asks for email on initial startup.  You can handle this by starting the server with the following command: 

```bash
docker run -p 9080:9080 -e ROS_TOS_EMAIL_ADDRESS=<your-email-address> realm/realm-object-server:latest
```

```bash
$ docker run -p 9080:9080 -e ROS_TOS_EMAIL_ADDRESS=<your-email-address> -v $(pwd)/data:/data realm/realm-object-server:latest
```

{% hint style="info" %}
This does not create an account in ROS with your email address.  You will need to register your users programmatically or via Realm Studio 
{% endhint %}

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

