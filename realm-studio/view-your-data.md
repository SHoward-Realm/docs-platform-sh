# View Your Data

## Realm Database

Realm Studio allows you to open up a local Realm file and view the data. Once opened, you can edit the data and it will save changes into the Realm.

{% hint style="info" %}
If you are working with Realm files on a device, you will need to transfer them onto your machine.
{% endhint %}

![Click &quot;Open Realm file&quot; to bring up the file window](../.gitbook/assets/image%20%2812%29.png)

![View and edit local Realm files - a demo file is available to try it out!](../.gitbook/assets/image%20%2822%29.png)

### Import Data

Realm Studio supports the ability to create a Realm file from CSV. To do so, go to `File` then `Create Realm from` --&gt; `CSV`.

{% hint style="warning" %}
Note that when importing from CSV, the first row will be used as the object properties.
{% endhint %}

![](../.gitbook/assets/image%20%283%29.png)

For example, a CSV formatted like: 

{% code-tabs %}
{% code-tabs-item title="data.csv" %}
```text
device,number,flag
gizmo,1,TRUE
widget,2,FALSE
```
{% endcode-tabs-item %}
{% endcode-tabs %}

This will result in a model named `data`that has three properties: `device` which will be a `string`, `number` which will be an `int` and `flag` which will be a `bool`

## Realm Platform

Realm Studio allows you to connect to a Realm Object Server, including Self-Hosted or Cloud instances. Once connected you can browser all of the synchronized Realms on the server and open them to view the data. Any changes you make to the data while viewing, will be automatically synchronized to any other devices sharing the data.

### Realms

Once you have logged into your server, you will be able to see a list of the available Realms:

![View the list of Realms in your server](../.gitbook/assets/image%20%285%29.png)

{% hint style="info" %}
You can create a new synchronized Realm. This is useful when you are creating a shared Realm that will be access by many users.
{% endhint %}

![View your data](../.gitbook/assets/image%20%2810%29.png)

### Edit Schema

With synchronized Realms, you can make changes to the schema. This is limited to additive changes only: creating new models and properties.

![](../.gitbook/assets/image%20%284%29.png)

![](../.gitbook/assets/image%20%2821%29.png)

### \_\_ Models

When using query-based synchronization, you'll notice that a number of special models are created inside of your Realm.  They are prefaced with "\_\_" and are used for implementation purposes.  These models are as follows: 

* \_\_Class
* \_\_DefaultRealmVersion
* \_\_Permission
* \_\_Realm
* \_\_Role
* \_\_User

Not what you were looking for? [Leave Feedback](https://realm3.typeform.com/to/A4guM3) 

